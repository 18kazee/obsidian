- profilesコントローラーの作成
- avatar uploaderを作成し、ユーザーと紐付け
- routesの設定
- viewの作成

## profilesコントローラーの作成

```shell
$ bin/rails g controller profiles edit update
```

```ruby
# profiles_controller
class ProfileController << ApplicationController
	before_action set_user, only: %i[edit update]

	def edit; end

	def update
		if @user.update(user_params)
			redirect_to profile_path, success: t('defaults.message.update', item: User.model_name.human)
		else
			flash.now[:danger] = t('defaults.message.not_update', item: User.model_name.human)
			render :edit
		end
	end
		
	def show; end

	private
	
	def set_user
		@user = User.find(current_user.id)
	end

	def user_params
		params.require(:user).permit(:email, :first_name, :last_name, :avatr, :avatar_cache)
	end
```

set_userについて参照すること <<  [[set_user]]


## avatar uploaderの作成

### avatarカラムの作成(userに紐付け)
```shell
$ bin/rails g migration AddAvatarToUsers avatar:string
```

### uploaderの作成
```shell
$ bin/rails g uploader avatar
```

##### userモデルに紐づけ
```ruby
# user.rb
mount_uploader :avatar, AvatarUploader
```

##### uploader/avatar_uploader.rb
```ruby
def default_url 
	'sample.jpg' 
end 

def extension_allowlist 
	%w(jpg jpeg gif png) 
end
```

## routesの設定

```ruby
#routes.rb
resoucce profile, only:[edit update show]
```

## viewの作成

```ruby
# profiles/show.erb
<%= content_for(:title t('.title')) %>
<div class="container, pt-3">
	<div class="row">
		<div class="col-md-10, offset-md-1">
			<h1 class="float-left, mb-5"><%= t('.title') %></h1>
			<%= link_to t('defaults.edit'), edit_profile_path, class: 'btn btn_success float-right' %>
			<table class="table">
				<tr>
					<th scope="row"><%= t(User.human_attribute_name(:email)) %></th>
					<td><%= current_user.email %></td>
				</tr>
				<tr>
					<th scope="row"><%= t(User.human_attribute_name(:full_name)) %></th>
					<td><%= current_user.decorate.full_name %></td>
				</tr>	
				<tr>
					<th scope="row"><%= t(User.human_attribute_name(:avatar)) %></th>
					<td><%= image_tag current_user.avatar.url, class: 'rounded-circle', size: '50x50' %></td>
				</tr>
			</table>
		</div>
	</div>
</div>
```

```ruby
# profiles/edit.erb
<%= content_form(:title, t('.title')) %>
<div class="container">
	<div class="row">
		<div class="col-md-10 offset-md-1">
			<h1><%= t(.title) %></h1>
			<%= form_with model: @user, url: profile_path, local: true do |f| %>
				<%= render 'shared/error_messages', object: f.object %>
				<%= f.label :email %>
				<%= f.email_field :email, class: 'form-control mb-3' %>
			
				<%= f.label :last_name %>
				<%= f.text_field :last_name, class: 'form-control mb-3' %>
			
				<%= f.label :fast_name %>
				<%= f.text_field :fast_name, class: 'form-control mb-3' %>
			
				<%= f.label :avatar %>
				<%= f.file_field :avatar, class: 'form-control mb-3', accept: 'image/*', onchange: 'previewFileWithId(preview)' %>
				<%= f.hidden_field :avatar_cache %>
			
				<div class="mt-3 mb-3">
					<%= image_tag @user.avatar.url,
												class: 'rounded-circle',
												id: 'preview',
												size: '100x100' %>
				</div>
				<%= f.submit 'btn btn-primary' %>
			<% end %>
		</div>
	</div>
</div>
```

##### コメントのavatarとヘッダーのavatar変更
```ruby
# comments/_comment.html.erb
<%= image_tag comment.user.avatar_url, class: 'rounded-circle', size: '50x50' %>
```

```ruby
<li class="nav-item dropdown dropdown-slide">
	<a href="#" class="nav-link" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" id="header-profile">
		<%= image_tag current_user.avatar_url, size: '40x40', class: 'rounded-circle mr15'%>
	</a>
	<div class="dropdown-menu dropdown-menu-right">
		<div class="dropdown-item"><%= current_user.decorate.full_name %></div>
		<div class="dropdown-divider"></div>
		<%= link_to (t 'profiles.show.title'), profile_path, class: 'dropdown-item' %>
		<%= link_to (t 'defaults.logout'), logout_path, class: 'dropdown-item', method: :delete %>
	</div>
</li>
```