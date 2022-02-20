# SES

## 概要

SESはAmazon Web Service提供されるメール送信サービス。SESを利用したメール送信については、WEBインターフェースを用いたテスト送信、AWS SDKを利用した送信、SMTPプロトコルで送信の3方法がある。

## 前提知識

* production利用の申請をしないと商用利用ができない
* 送信元メールアドレス(or ドメイン)は、verificationプロセスを実施してverifiedとして登録しておく必要がある
  * ＝ Domain Verification Record(SESが送信元サーバを検証するために利用)をDNSに登録しておく必要がある
* バウンスメールの処理(エラーとなったメールアドレスには再送信しないなど)を適切に実装し、バウンス率を低く抑えないとサービス停止となる
* 逆引きDNS設定は不要

## SPF、DKIM、Domain Verification Record、IAM、MXレコード

[Linux/Postfix/Send.md](/Linux/Postfix/Send.md)を参照

## バウンス処理

バウンスメールの処理をする方法として、SESのバウンス通知(Notification)でAamazon SNSサービスを利用する方法を記載する。

### 手順(Webサーバー)

SESからのバウンス通知(JSONデータ)を受け取り、エラー情報(メールアドレス等)を保存する処理を作成する。以下はPHPでAWS SDKを利用した実装例。

※ここでは、https://example.com/ses/error/をエンドポイントとする。

        try
    	{
	    	// POSTデータ(JSON)を取得
	    	$data = json_decode($this->app->request()->getBody(), true);
	
	    	if (empty($data) || !is_array($data))
	    	{
	    		throw new AeonException(Code::ERR_PARAMETER, Msg::ERR_INVALID_DATA);
	    	}
	    	
	    	// SNS Messageオブジェクト生成
	    	$snsMsg = Message::fromArray($data);
	    	
	    	// SNS MessageValidator生成
	    	$snsValidator = new MessageValidator();
	    	
	    	// バリデーション
	    	$snsValidator->validate($snsMsg);
	    	
	    	// 初回のSubscription確認要求
	    	if ($snsMsg->get('Type') === 'SubscriptionConfirmation')
	    	{
	    		(new Client())->get($snsMsg->get('SubscribeURL'))->send();
	    	}
	    	// 通知
	    	else if ($snsMsg->get('Type') === 'Notification')
	    	{
		    	// バウンスの詳細を取得
		    	$detail = json_decode($snsMsg->get('Message'));
		    	
		    	// 受信者情報リスト
		    	$recipients = array();
		    	// Bounce
		    	if ($detail->notificationType === NotificationType::BOUNCE)
		    	{
		    		$recipients = $detail->bounce->bouncedRecipients;
		    	}
		    	// Compaint
		    	else if ($detail->notificationType === NotificationType::COMPLAINT)
		    	{
		    		$recipients = $detail->complaint->complainedRecipients;
		    	}
		    	
		    	// 受信者情報が抽出できなければ無視
		    	if (empty($recipients))
		    	{
		    		return;
		    	}
		    	
		    	// エラーとなったメールアドレスリスト
		    	@$mailList = array_map(function($each) { return $each->emailAddress; }, $recipients);
		    	
		    	// DB接続
		    	parent::connectDb();
		    	
		    	// エラーメールアドレスマッパー生成
		    	$errorMailMapper = new ErrorMailaddressMapper($this->dbh);
		    	
		    	foreach ($mailList as $each):
		    		$dto = new ErrorMailaddressModel();
		    		$dto->hash = hash_hmac('sha256', $each, MAIL_HASH_SALT);
		    		
		    		$errorMailMapper->insert($dto);
		    	endforeach;
	    	}
    	}
    	catch (AeonException $e)
    	{
			LogUtil::dumpDebug($e->asString());
			$this->app->response->setStatus(403);
    	}
    	catch (Exception $e)
    	{
    		$wrap = new AeonException(Code::ERR_UNKNOWN, Msg::ERR_UNKNOWN, $e);
			LogUtil::dumpError($wrap->asString());
			$this->app->response->setStatus(403);
    	}

>Notificationは、UAがAmazon Simple Notification Service Agentで通知されるが、IPアドレスは一定ではないため、セキュリティポリシーでのIP制限は難しい

### 手順(SNS)

SNSのダッシュボードを開き、Create New Topicから新規のTopicを作成する。

作成したTopicに新たなSubscriptionを作成し、以下の通り設定する。

| 項目 | 設定値 |
| - | - |
| Protocol | HTTPS |
| Endpoint | https://example.com/ses/error/ | 

### 手順(SES)

SESののDomain設定から対象のドメインのNotifications設定を変更する。設定値は以下の通り。

| 項目 | 設定値 | 説明 |
| - | - | - |
| Bounces | 作成したTopic | bounceを通知するか |
| Complaints | 作成したTopic | complaintを通知するか |
| Deliveries | No SNS topic | 通常の配信結果を通知するか |
| Email Feedback Forwarding | Disabled | bounceやcomplaintを送信元メールアドレスへメールで送信するか |