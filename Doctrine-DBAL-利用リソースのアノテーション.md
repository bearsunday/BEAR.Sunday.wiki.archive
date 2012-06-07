    /**
     * @Pager
     */

ページャー使用

    /**
     * @Order(default="+", control={"db_id", "query_id"})
     */
order指定

default=デフォルト順序
control = 順序指定可。１）DBコラム名, ２）GETクエリーキー名


    /**
     * @Limit(10)
     */
アイテム数指定
リミットクエリーまたは@Pagerと組み合わせてPagerの１ページのアイテム数


    /**
     * @SqlMap(id=plain_list, select="id,name,age") 
    */
SQLマップ