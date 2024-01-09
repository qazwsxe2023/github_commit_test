
■Eclipse 2018.09 をインストール
pleiadesのJavaの Full Edition を入れる
http://mergedoc.osdn.jp/
バージョンは 2018.09 でないと Jacoco のレポート作成時にエラーが出る


■Amazon Corretto 8 をEclipseのデフォルトJREに設定
1.Amazon Corretto 8をダウンロード
https://aws.amazon.com/jp/corretto/

2.EclipseのデフォルトJREに設定
「ウィンドウ」
->「設定」
->「インストール済みのJRE」
->「追加」
->インストールした「\Amazon Corretto\jre8」のディレクトリを指定
->追加したjre8をデフォルトに指定
->「適用して閉じる」


■Eclipseにプロジェクトをインポート
1.「ファイル」
->「インポート」
->「Gradle->既存のGradleプロジェクト
->ルートにこのディレクトリを指定
->「ワークスペース設定を上書き」にチェックを入れる
->「Gradleディストリビューション->Gradleバージョンの指定」で4.10.3を選択
-> 「完了」


■jarのビルド方法
1.どこかにGradleタスクタブがあるので探す。
2.なかったら「ウィンドウ」->「ビューの表示」から追加
3.ビルドしたいプロジェクトを選択
4.「build->jar」をダブルクリックでプロジェクトフォルダ/jar内にjarファイルが作成される



■設定
・まず環境変数を見に行って、なければsetting.propertiesから取得する
・配信時は設定ファイルを含めなければ、環境変数からしか見ない
・ユニットテスト時は「src/test/resources/setting.properties」に設定を書けばOK


■ログ出力
・自前でCloudWatchに書き出している
・ローカルではsetting.propertiesにCloudWatchの設定を書かなければ、標準出力に出す


■ユニットテスト
・JUnitとMockito使う

・DynamoDBLocal
https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html
↓起動コマンドこんな感じ
java -Djava.library.path="C:/Program Files (x86)/DynamoDBLocal-2015-07-16/DynamoDBLocal_lib" -jar "C:/Program Files (x86)/DynamoDBLocal-2015-07-16/DynamoDBLocal.jar" -sharedDb

・Redis
\\10.25.20.5\TelemaFile2\000011_ツール\redisbin


■カバレッジ
・Gradleタブで対象のプロジェクトの「other->report」でテスト実行とレポートが作成される
	プロジェクト\build\reports\jacoco\test\html


■ソースチェック
・Gradleタブで対象のプロジェクトの「other->checkstyleMain」でソースのチェック結果が作成される
	プロジェクト\build\reports\checkstyle



■配置（CommonLibrary）
src
├ main
│	├ java
│	│	└ jp.tscgateway.common_library
│	│		└ (略)		ユーティリティクラス群
│	└ resources	未使用
└ test
	├ java
	│	└ jp.tscgateway.common_library		ユーティリティクラス群のユニットテスト
	｜		├ XxxNoReportTest.java		実際にDynamoとかS3を使うテスト。このクラスはカバレッジから外す
	│		└ XxxTest.java				カバレッジ用のユニットテスト。PowerMockito使ってエラー系も通す
	└ resources	未使用


■配置（TestLibrary）	test時のみクラスパスに追加
src
└ main
	├ java
	│	├ lambda.test
	│	│	└ TestContext.java		Lambdaクラスのローカルテストに使用。本来はLambdaが渡してくる実行時のパラメータを、疑似的に作る用
	│	└ shared.test
	│		└ (略)		ユニットテストに使うMatcherの拡張クラス
	└ resources	未使用


■配置（Lambda用）
src
├ main
│	├ java
│	│	└ jp.tscgateway.xxx
│	│		├ Config.java				環境変数から取得した設定を保持する
│	│		├ XxxLambda.java			ファイル一件分ごとにServiceクラスを実行する。このクラスの実行は面倒なので、業務ロジック部分はServiceクラスに分けるのを推奨
│	│		└ XxxService.java			業務ロジックを書く。ファイル一件分の処理に相当
│	└ resources
│		└ setting.properties		環境変数に持たせたくない設定はここに書く。そんな設定はないのなら、ファイル自体なくてよい
└ test
	├ java
	│	└ jp.tscgateway.xxx
	｜		├ XxxLambdaNoReportTest.java	実際にDynamoとかS3を使うテスト。このクラスはカバレッジから外す
	│		├ XxxLambdaTest.java			Lambdaクラスのユニットテスト。PowerMockito使うと楽
	│		└ XxxServiceTest.java			Serviceクラスのユニットテスト。Manager系のクラスはMockito使うと楽
	└ resources
		└ setting.properties		ユニットテスト時に使用する設定を書いておく


