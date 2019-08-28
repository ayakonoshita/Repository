/**
 * バリデーションパスの指定
 * @validate konoshita-a_task/validator#validateRule
 * @validate konoshita-a_task/validator#validateRule2
 * @validate konoshita-a_task/validator#validateRule3
 * 入力チェック時に実行する関数の指定
 * @onerror handleErrors
 */

function init(request) {
    //ユーザプロファイル情報を取得
    var userProfile    = Contexts.getUserContext().userProfile;
        
    //要素を格納する配列
    var published = [request.published, request.published2, request.published3];
    var title = [request.title, request.title2, request.title3];
    
    //取得した値を格納する配列
    var createList = [];

    //タイトルが送信されたリクエストの値のタイトルと公開フラグの値のみ格納
    for(var i = 0; i < title.length; i++){
    	// タイトルが入力されていない場合処理を行わない
    	if( title[i] == "" ) continue;
    	createList[i] = [title[i], published[i]];
    }
    
   //繰り返し処理で使用する変数の宣言
	var resultObject; //トランザクション結果取得用
	var tit;//タイトル取得用
	var pub;//公開フラグ変換用

	for(var i = 0; i < createList.length; i++){ 
		//タイトルの値を取り出す
		tit = createList[i][0];
		
		//公開フラグの値の取り出し・変換
    	if(createList[i][1] == null){
    		pub = true;
    	}else{
    		pub = false;
    	}

		//格納データの作成
	    var insertObject = {
	    	id:Identifier.get(),//ID　ユニークな値を設定
	    	title:tit,//タスク名　画面から取得
	    	status:0,//ステータス　未完成
	    	published:pub,//公開フラグ　公開
	    	user_cd:userProfile.userCd,//登録ユーザー　ログインユーザーから取得
	    	created_at:new Date(),//登録日時　現在日時
	    	updated_at:new Date()//更新日時　現在日時
	    };
	   
	    //SQL実行のトランザクション開始
	    Transaction.begin(function() {
	    	// insert SQLの実行
	 	    var result = new TenantDatabase().insert("task", insertObject);
	 	    
 			//エラー処理
	        if(result.error){//エラー時
	        	//ロールバック
	            Transaction.rollback();
	            //メッセージ
	            resultObject = {
	                error:true,
	                errorMessage:"データ登録時にエラーが発生しました。",
	                detailMessages:["管理者にお問い合わせください。"]
	            };
	        } else {//成功時
	            resultObject = {
	                error:false,
	                errorMessage:"",
	                successMessage:"登録が完了しました。"
	            };
	        }
	    });
	}
    //結果を送信
    var response = Web.getHTTPResponse();
    response.setContentType('application/json; charset=utf-8');
    response.sendMessageBodyString(ImJson.toJSONString(resultObject));

	//入力チェック時に実行する関数
	function handleErrors(request, validationErrors) {
		Transfer.toErrorPage({
		    title: 'エラー',
		    message: '入力チェックエラーが発生しました。',
		    detail: validationErrors.getMessages()
	    });
	}
}
