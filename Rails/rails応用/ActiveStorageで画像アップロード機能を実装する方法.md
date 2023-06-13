
# [](https://qiita.com/mmaumtjgj/items/6e6ce55d4b80a16577eb#activestorage%E3%81%A8%E3%81%AF)ActiveStorageとは

`Active Storage`は、アプリケーションにアップロードした画像の変形や、PDFや動画などの画像以外のアップロードファイルの内容を代表する画像の生成、任意のファイルからのメタデータ抽出に利用できる。

> Active Storageは、Amazon S3、Google Cloud Storage、Microsoft Azure Storageなどのクラウドストレージサービスへの**ファイルのアップロードや、ファイルをActive Recordオブジェクトにアタッチする機能を提供**します。 development環境とtest環境向けのローカルディスクベースのサービスを利用できるようになっており、ファイルを下位のサービスにミラーリングしてバックアップや移行に用いることも可能です。

筆者は、画像アップロードする系のものを、`CarrierWave`で学習したので、２つの違いを見てみる。

#### [](https://qiita.com/mmaumtjgj/items/6e6ce55d4b80a16577eb#%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AE%E8%A6%B3%E7%82%B9%E3%81%8B%E3%82%89%E8%A6%8B%E3%82%8B)バリデーションの観点から見る

`Active Storage`を使用する際にバリデーションエラーが起きると、5.2系〜6.0系以前では　`model`のインスタンスの`attribute`に代入した時点で、アップロードしたファイルがストレージに保存されてしまい、その結果として、`blob`にエラー用の一時データが溜まってしまう可能性があった。（この状況を解決するためには、一時データを削除する仕組みを別途作ってあげる必要が生じる。）

一方、`CarrierWave`にはこのような仕組みはなく、エラーが起きた段階でファイルがストレージに保存されることにはならないので、考える必要はない。

ただ、rails6.0へのアップデートで`Active Storage`の機能変更があり、`saveメソッド`が実行された後に、ストレージに保存される仕様に変更されたことで`blobデータ`にエラーデータが溜まることは無くなったので、6.0以降であれば、問題なく利用できると考えらる。

#### [](https://qiita.com/mmaumtjgj/items/6e6ce55d4b80a16577eb#%E3%82%AF%E3%83%A9%E3%82%A6%E3%83%89%E3%82%B9%E3%83%88%E3%83%AC%E3%83%BC%E3%82%B8%E3%81%A8%E3%81%AE%E9%80%A3%E6%90%BA%E3%81%8B%E3%82%89%E8%A6%8B%E3%82%8B)クラウドストレージとの連携から見る

`Active Storage`設定の際にやるべきことをまとめると下記の内容が必要。

```
・Gemfileへの追記
・rails active_storage:installコマンドとマイグレーションの実行による
　blobテーブルとattachmentテーブルのそれぞれの作成
・config/storage.yml ファイルにおけるクラウド用の各種設定
・各種環境設定ファイル「configフォルダの中」へのconfig.active_storage.service変数の設定
```

`CarrierWave`では、「Amazon S3、Google Cloud Storage、Microsoft Azure Storage」などのクラウドストレージ対応gemのfogと連携させることでファイルのアップロードが可能になる。具体的にはらの内容が必要になる。

```
・Gemfileへの追記
・画像保存用のカラムを追加するためのマイグレーションファイルの生成
・マイグレーションの実行
・アップローダー用のモデルの作成
・モデルとアップローダーとの紐づけ
・config/initializers/carrierwave.rb を作成し、クラウド用の各種設定
```

既存のテーブルにカラムを追加するか、新規テーブルに対してのカラムを操作するかどうかで違いがあるようだ。

#### [](https://qiita.com/mmaumtjgj/items/6e6ce55d4b80a16577eb#active-storage%E3%81%A8carrierwave%E3%81%AE%E9%81%95%E3%81%84%E3%81%BE%E3%81%A8%E3%82%81)Active StorageとCarrierWaveの違いまとめ

・`Active Storage`は基本的な機能は揃っているので、**シンプルにファイルアップロード機能を実装するのには適している**（Rails6.0系以降であれば）。
・`CarrierWave`での実装の際には**各種細かい設定ができるので、細かいカスタマイズが必要な場合**には`CarrierWave`が良さそう。

## 実装内容

現状`favicon````og:image`の２つの削除機能がついていない、画像投稿機能がある。

ここに`main images`という複数画像投稿機能をもう１つ増やしつつ、それぞれ画像削除機能も追加していく。

なお、画像削除の処理は`params`で条件分岐をせずに`ActiveStorage::attachment`から取得。理由は後述にて。

## 実装の流れ

1.`site`モデル、バリデーターの編集  
2.`siteコントローラー`の編集  
3.`destroy`メソッドの追加  
4.`view`の追加

### siteモデルの編集

まずは`main images`のカラムに追加や、そのバリデーションを追記する。

models/site.rb
```ruby
class Site < ApplicationRecord
  has_one_attached :og_image
  has_one_attached :favicon

  has_many_attached :main_images
　　　#↑ここと
  validates :name, presence: true, length: { maximum: 100 }
  validates :subtitle, length: { maximum: 100 }
  validates :description, length: { maximum: 400 }
  validates :og_image, attachment: { purge: true, content_type: %r{\Aimage/(png|jpeg)\Z}, maximum: 524_288_000 }
  validates :favicon, attachment: { purge: true, content_type: %r{\Aimage/png\Z}, maximum: 524_288_000 }
  validates :main_images, attachment: { purge: true, content_type: %r{\Aimage/(png|jpeg)\Z}, maximum: 524_288_000 }
　　#↑ここを追加
end
```

次にカスタムバリデーターを編集する。

> **カスタムバリデーターとは**  
> モデルとは別で、自分でバリデータやバリデーションメソッドを作成することができる。app/配下にvalidatorsディレクトリ(app/validators)を作成することで、自動で読み込んでくれる。

app/validators/attachment_validator.rb
```ruby
class AttachmentValidator < ActiveModel::EachValidator
  include ActiveSupport::NumberHelper

  def validate_each(record, attribute, value)
    return if value.blank? || !value.attached?

    has_error = false

　　　　　　　　#↓画像サイズ
    if options[:maximum]
      if value.is_a?(ActiveStorage::Attached::Many)
　　　　　　　　　　　　　#　↑複数の画像が保存された時
        value.each do |v|
          unless validate_maximum(record, attribute, v)
            has_error = true
            break　#ループから抜け出す。falseが出たら次のvalueに行かずエラー出す。
          end
        end
      else
        has_error = true unless validate_maximum(record, attribute, value)
　　　　　　　　　　　　　　　　　#↑一枚の時
      end
    end

　　　　　　　　#↓画像タイプ
    if options[:content_type]
      if value.is_a?(ActiveStorage::Attached::Many)
        value.each do |v|
          unless validate_content_type(record, attribute, v)
            has_error = true
            break
          end
        end
      else          
        has_error = true unless validate_content_type(record, attribute, value)
      end
    end

    record.send(attribute).purge if options[:purge] && has_error
  end

  private

  def validate_maximum(record, attribute, value)
　　　　　　　#↓bite_sizeがmaximumより大きかったらエラーを末尾に追加する
    if value.byte_size > options[:maximum]
      record.errors[attribute] << (options[:message] || "は#{number_to_human_size(options[:maximum])}以下にしてください")
      false
    else
      true
    end
  end

  def validate_content_type(record, attribute, value)
    if value.content_type.match?(options[:content_type])
      true
    else
      record.errors[attribute] << (options[:message] || 'は対応できないファイル形式です')
      false
    end
  end
end
```

### siteコントローラーの編集

ファイルをアップロードした時に、`params`でちゃんと送れるようコントローラーを編集する。

sites_controller.rb
```ruby
class Admin::SitesController < ApplicationController
  layout 'admin'

  before_action :set_site

  def edit
    authorize(@site)
  end

  def update
    authorize(@site)

    if @site.update(site_params)
      redirect_to edit_admin_site_path
    else
      render :edit
    end
  end

  private

  def site_params
    params.require(:site).permit(:name, :subtitle, :description, :favicon, :og_image, main_images: [])
 #ここに　main_images: []を追加。複数画像を登録できるように注意。
  end

  def set_site
    @site = Site.find(current_site.id)
  end
end
```

### destroyメソッドの追加

・画像削除の処理は`params`で条件分岐をせずに`ActiveStorage::attachment`から取得

この理由は、もし`params`を用いると、
`@site.favicon.purge if params[:status] == 'favicon'`　
という感じで全ての要素に対して実装しなければならなくなる。
また`active_storage_attachments`テーブルには`record_type`カラムがある（=idが一意になっている）ので、これを利用して削除対象のリソースを取得する。  
▷[Railsガイド/カスタムバリデーションを実行する](https://railsguides.jp/active_record_validations.html#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%90%E3%83%AA%E3%83%87%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E5%AE%9F%E8%A1%8C%E3%81%99%E3%82%8B)

#####  ルーティングに画像削除のネストを追加する

routes.rb
```ruby
resource :site, only: %i[edit update] do
      resources :attachments, controller: 'site/attachments', only: %i[destroy]
end
```

##### `site/attachments`通りにコントローラー内にファイルを作成し記入する。

app/controllers/admin/site/attachments_controller.rb
```ruby
class Admin::Site::AttachmentsController < ApplicationController
  def destroy
    authorize(current_site)
    image = ActiveStorage::Attachment.find(params[:id])
    image.purge
    redirect_to edit_admin_site_path
  end
end
```

`ActiveStorage`の削除コマンドは`destroy`ではなく、`purge`を使用する。  
▷[Railsガイド/ファイルを削除する](https://railsguides.jp/active_storage_overview.html#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92%E5%89%8A%E9%99%A4%E3%81%99%E3%82%8B)

ただこのままでは誰でも削除できてしまうので、管理者（admin）だけ削除できるようにする。なお`Pundit`を導入済みなので簡単にできる。  
▷[【Rails】authorizeとは？[Pundit]](https://qiita.com/mmaumtjgj/items/c7fc40619a15cce5ccfc)

app/policies/site_policy.rb
```ruby
def destroy?
  user.admin?
end
#上記を追記
```

### viewの追加

後はもう編集フォームに新たに複数画像投稿ろ削除ボタンを追加するだけで完成する。

sites/edit.html.slim
```ruby
.box.box-primary
  = simple_form_for [:admin, @site], url: admin_site_path do |f|
    .box-body
      = f.error_notification
      = f.input :name
      = f.input :subtitle
      = f.input :description, as: :text
      = f.input :favicon, as: :file, hint: 'PNG (32x32)'

      - if @site.favicon.attached?
        = image_tag @site.favicon_url('32x32')
        br
        br
        = link_to '削除', admin_site_attachment_path(@site.favicon.id), class: %w[btn btn-danger], method: :delete
      　 /↑ここと

      = f.input :og_image, as: :file, hint: 'JPEG/PNG (1200x630)'

      - if @site.og_image.attached?
        = image_tag @site.og_image_url(:ogp), class: 'img-responsive'
        br
        br
        = link_to '削除', admin_site_attachment_path(@site.og_image.id), class: %w[btn btn-danger], method: :delete
　　　　　　　　　　　　　　　　　/↑ここと
　　　　　　　　　　　　　　/↓ここを追加
      = f.input :main_images, as: :file, input_html: { multiple: true }, hint: 'JPEG/PNG (1200x400)'

      - if @site.main_images.attached?
        .main_images_box
          - @site.main_images.each do |main_image|
            .main_image
              = image_tag main_image.variant(resize:'300x100').processed
              br
              = link_to '削除', admin_site_attachment_path(main_image.id),method: :delete, class: 'btn btn-danger'

    .box-footer
      = f.button :submit, '保存', class: %w[btn btn-primary]
```

https://qiita.com/mmaumtjgj/items/6e6ce55d4b80a16577eb