親モデルに属する子モデルのインスタンスを新たに生成したい場合に使うメソッド。  
（親モデルと子モデルは、アソシエーション設定あり）  
外部キーに値が入った状態でインスタンスが生成できる。

ブログに紐づくコメントインスタンスを生成したい場合  
※BlogモデルとCommentモデルは、１対多のアソシエーションを設定しているとします。

```ruby
@blog = Blog.find(params[:id])
@comment = @blog.comments.build  ## 親モデル.子モデル.buildという形式
```

生成されるコメントインスタンスの中身

```shell
$ @comment
 id: nil,
 blog_id: 1,
 user_id: nil,
 content: nil,
 created_at: nil,
 updated_at: nil
```

blog_idに値が入っています。