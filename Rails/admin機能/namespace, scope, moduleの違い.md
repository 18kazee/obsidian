
Railsのroutingにおけるscope / namespace / module の違いについて

scope = URL「指定のパスにしたい」・ファイル構成「変えたくない」
namespace = URL「指定のパスにしたい」・ファイル構成「指定のパスにしたい」
module = URL「変えたくない」・ファイル構成「指定のパスにしたい」

----
### scope
以下の場合はscopeが望ましい

-   URLは指定のパスにしたい
-   ファイル構成変えたくない
```ruby
Rails.application.routes.draw do
	scope :blog do 
		resources :articles 
	end
end
```

----
### namespace
以下の場合はnamespaceが望ましい

-   URLは指定のパスにしたい
-   ファイル構成も指定のパスにしたい
```ruby
Rails.application.routes.draw do 
	namespace :blog do 
		resources :articles 
	end 
end
```

----
### module
以下の場合はmoduleが望ましい

-   URLは変えたくない
-   ファイル構成だけ指定のパスにしたい
```ruby
Rails.application.routes.draw do 
	scope module: :blog do 
		resources :articles 
	end 
end
```

----

>参照
>https://qiita.com/ryosuketter/items/9240d8c2561b5989f049