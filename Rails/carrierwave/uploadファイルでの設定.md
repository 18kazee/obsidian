
```ruby
#upload

class BoardImageUploader < CarrierWave::Uploader::Base
  storage :file

  def store_dir
    "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
  end

  def default_url
    'board_placeholder.png'
  end

  def extension_whitelist
    %w[jpg jpeg gif png]
  end
end
```


### store_dir

carrierwaveを通じた画像のアップロード先をどこにするのかを指定しており、
指定されたディレクトリにアップロードされたファイルが保存されていくことになります。
DBに保存されるのは、ファイルそのものではなく、ファイルへの参照データになります。

```ruby
def store_dir
  "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
end
```


### default_url

デフォルトでの画像を指定しています。  
プレビューでは最初にここで指定した画像が表示されます。

```ruby
def default_url  
'board_placeholder.png'  
end
```


### extension_whitelist 

アップロードできる拡張子を制限します。

```ruby
def extension_whitelist  
%w[jpg jpeg gif png]  
end
```
