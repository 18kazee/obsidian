

### articleモデルにscopeを追加して短いクエリを使用したメソッドを使用作成

models/article.rb
```ruby
belongs_to :category
belongs_to :author
has_many :article_tags
has_many :tags, through: :article_tags
has_many :sentences, through: :article_blocks, source: :blockable, source_type: 'Sentence'

scope :viewable, -> { published.where('published_at < ?', Time.current) }
scope :new_arrivals, -> { viewable.order(published_at: :desc) }
scope :by_author, ->(author_id) { where(author_id: author_id) }
scope :by_category, ->(category_id) { where(category_id: category_id) }
scope :by_tag, ->(tag_id) { joins(:article_tags).where(article_tags: { tag_id: tag_id }) }
scope :title_contain, ->(word) { where('title LIKE ?', "%#{word}%") }
scope :body_contain, ->(body) { joins(:sentences).merge(where('sentences.body LIKE ?', "%#{body}%")) }
scope :past_published, -> { where('published_at <= ?', Time.current) }

```

scopeのby_author, by_category, by_tagはセレクトで選択できるように作成します。
title_containとbody_containscは文字列から検索出来るように作成します。

##### by_tag
Active Recordでは、`joins`メソッドを使用して関連付けで`JOIN`句を指定する際に、モデルで定義された関連付けの名前をショートカットとして使用できます。by_tagスコープは、引数として渡された `tag_id` に一致する `article_tags` テーブルのレコードを取得し、それに関連付けられた `Article` モデルのレコードを取得するために、`joins` メソッドを使用しています。

##### body_contain
`sentences`テーブルと`articles`テーブルを結合（`joins`）し、`sentences`テーブルの`body`カラムに指定された`body`文字列が含まれるレコードを取得するために、`where`句を使って検索条件を指定しています。`%`は部分一致を表します。

また`.merge` メソッドは、`joins` などで生成された `ActiveRecord::Relation` オブジェクトに対して、その後にチェインすることで、新たな検索条件を追加するために使用されます。

このコードの場合、`joins(:sentences)` によって、 `Sentences` テーブルと `Article` テーブルが内部結合されています。そして、 `where('sentences.body LIKE ?', "%#{body}%")` によって、 `Sentences` テーブルから `body` カラムが `body` 引数に部分一致するレコードを取得するための検索条件を生成しています。

しかし、この時点では、 `Article` テーブルに対して検索条件が追加されていないため、 `merge` メソッドを使用して、`Sentences` テーブルと `Article` テーブルの両方に検索条件を適用することができます。これにより、`Article` テーブルから、 `Sentences` テーブルの条件に基づいて、関連する記事を取得することができます。

### ActiveModelを使用してDBに依存しないモデルの作成

app/forms/serch_form_atricles.rb
```ruby
class SearchArticlesForm

	include ActiveModel::Model
	include ActiveModel::Attributes

	attribute :author_id, :integer
	attribute :category_id, :integer
	attribute :tag_id, :integer
	attribute :title, :string
	attribute :body, :string

	def search
		relation = Article.distinct
		
		title_words.each do |word|
			relation = relation.title_contain(word)
		end
		relation = relation.by_author(author_id) if author_id.present?
		relation = relation.by_tag(tag_id) if tag_id.present?
		body_words.each do |word|
			relation = relation.body_contain(word)
		end
		relation
	end

	def title_words
		title.present? ? title.split(nil) : []
	end
  
	def body_words
		body.present? ? body.split(nil) : []
	end
 end
```

ActiveModel::Modelをincludeすることで、Railsのモデルと同様の機能を、
ActiveModel::Attributesをincludeすることで、属性の定義や属性の初期化などができるようになります。

例えば、ActiveRecordを使用していないRailsアプリケーションにおいて、ActiveModel::Modelを使うことで、モデルに必要な機能（バリデーションやコールバックなど）を実装することができます。また、ActiveModel::Attributesを使用することで、Active Recordのように属性に型やバリデーションを定義できます。このモジュールを使用すると、属性の値の取得と設定にActive Recordと同じようにメソッドを使用できます。また、フォームオブジェクトのような場合に、Strong Parametersを使用せずに属性の設定ができます。

### コントローラーの作成

app/controllers/admin/articles_controleller.rb
```ruby
def index
　　authorize(Article)

　　@search_articles_form = SearchArticlesForm.new(search_params)
　　@articles = @search_articles_form.search.order(id: :desc).page(params[:page]).per(25)
end

# 中省略

private

def search_params
　　params[:q]&.permit(:title, :body, :category_id, :author_id, :tag_id)
end
```

### Viewの作成

```ruby

#前省略

= form_with model: @search_articles_form, scope: :q, url: admin_articles_path, method: :get, html: { class: 'form-inline' } do |f|
	=> f.select :category_id, Category.pluck(:name, :id), { include_blank: 'カテゴリ' }, class: 'form-control'
	=> f.select :author_id, Author.pluck(:name, :id) , { include_blank: '著者' }, class: 'form-control'
	=> f.select :tag_id, Tag.pluck(:name, :id), { include_blank: 'タグ' }, class: 'form-control'
	=> f.search_field :body, placeholder: '記事内容', class: 'form-control'
	.input-group
		= f.search_field :title, placeholder: 'タイトル', class: 'form-control'
		span.input-group-btn
			= f.submit '検索', class: %w[btn btn-default btn-flat]

# 以下省略
```

