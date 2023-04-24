タスクとユーザーのCRUD機能について、テストケースを作成

-   [RSpec]のセットアップ済 　
-   FactoryBot導入済
-   [CRUD]機能自体は[実装済]
-   ログイン認証はsorceryを使用

### 流れ
-   必要なgemの導入
-   SystemSpecファイルの作成
-   ドライバの設定
-   モジュールの設定
-   SystemSpecファイルの設定

### Gemの導入

Gemfile
```ruby
group :test do
  gem 'webdrivers' #ブラウザの挙動を確認
  gem 'capybara' #E2Eテスト用フレームワーク
end
```

spec/spec_helper
```ruby
require 'capybara/rspec'
```

### SystemSpecファイルの作成

```shell
% rails g rspec:system tasks
```
※必要に応じて、`users`と`user_sessions`も作成。

### ドライバの設定

> ドライバとは、Capybaraを使ったテスト/Specにおいて、ブラウザ相当の機能を利用するために必要なプログラムです。  
> 『現場で使える Ruby on Rails5 速習実践ガイド』P193

spec/support/capybara.rb
```ruby
RSpec.configure do |config|
  config.before(:each, type: :system)do
    driven_by(:selenium_chrome_headless) #ドライバを一元指定
  end
```

### モジュールの設定

今回、ログイン前・ログイン後に分けてテストを実施します。  
モジュールにログイン処理を設定しておきます。  
そうすれば、各Specで呼び込むことができるので、コードがすっきりします。  
`support`[ディレクト]リ内にします。

spec/support/login_macros
```ruby
module LoginMacros
  def login(user)
    visit login_path
    fill_in 'Email', with: user.email
    fill_in 'Password', with: 'password'
    click_button 'Login'
  end
end
```

モジュールを作っただけでは使用できないので、`include`する必要があります。

spec/rails_helper.rb
```ruby
  config.include FactoryBot::Syntax::Methods 
  config.include LoginModule #追記
```

そして`rails_helper.rb`を読み込むために、各Specファイルには、上部に`require 'rails_helper'`があります。

```ruby
require 'rails_helper'
```

最後にパスを通す。`spec/support`以下を読み込むように設定する。  
以下の[コメントアウト]を外して有効化します。

spec/rails_helper
```ruby
# The following line is provided for convenience purposes. It has the downside
# of increasing the boot-up time by auto-requiring all files in the support
# directory. Alternatively, in the individual `*_spec.rb` files, manually
# require only the support files necessary.
#
Dir[Rails.root.join('spec', 'support', '**', '*.rb')].sort.each { |f| require f } #コメントアウトを外す
```

### タグの設定

タグの機能を使うと、タグを設定したテストだけ実行してくれます。  
`:focus`タグを付けたテストが何もないときは、全てのテストを実行します。

spec/spec_helper
```ruby
# This allows you to limit a spec run to individual examples or groups
  # you care about by tagging them with `:focus` metadata. When nothing
  # is tagged with `:focus`, all examples get run. RSpec also provides
  # aliases for `it`, `describe`, and `context` that include `:focus`
  # metadata: `fit`, `fdescribe` and `fcontext`, respectively.
  config.filter_run_when_matching :focus #コメントアウトを外す。
```

### SystemSpecファイルの設定
spec/system/users.spec.rb
```ruby
require 'rails_helper'

RSpec.describe "Users", type: :system do
  let(:user) { create(:user) }
  let(:other_user) { create(:user) }

  describe 'ログイン前' do
    describe 'ユーザー新規登録' do
      context 'フォームの入力値が正常' do
        it 'ユーザーの新規登録が成功' do
          visit new_user_path
          fill_in 'Email', with: 'example@example.com'
          fill_in 'Password', with: 'password'
          fill_in 'Password confirmation', with: 'password'
          click_button 'SignUp'
          expect(page).to have_content("User was successfully created.")
          expect(current_path).to eq login_path
        end
      end

      context 'メールアドレスが未入力' do
        it 'ユーザーの新規作成が失敗する' do
          visit new_user_path
          fill_in 'Email', with: ''
          fill_in 'Password', with: 'password'
          fill_in 'Password confirmation', with: 'password'
          click_button 'SignUp'
          expect(page).to have_content '1 error prohibited this user from being saved'
          expect(page).to have_content "Email can't be blank"
          expect(current_path).to eq users_path
        end
      end

      context '登録済みのメールアドレスを使用する' do
        it 'ユーザーの新規登録が失敗する' do
          existed_user = create(:user)
          visit new_user_path
          fill_in 'Email', with: existed_user.email
          fill_in 'Password', with: 'password'
          fill_in 'Password confirmation', with: 'password'
          click_button 'SignUp'
          expect(page).to have_content '1 error prohibited this user from being saved'
          expect(current_path).to eq users_path
          expect(page).to have_content "Email has already been taken"
          expect(page).to have_field 'Email', with: existed_user.email
        end
      end
    end

    describe 'マイページ' do
      context 'ログインしていない状態' do
        it 'マイページへのアクセスが失敗する' do
          visit user_path(user)
          expect(page).to have_content "Login required"
          expect(current_path).to eq login_path
        end
      end
    end
  end

  describe 'ログイン後' do
    before { login(user) }

    describe 'ユーザー編集' do
      context 'フォームの入力値が正常' do
        it 'ユーザーの編集が成功する' do
          visit edit_user_path(user)
          fill_in 'Email', with: 'update@example.com'
          fill_in 'Password', with: 'update_password'
          fill_in 'Password confirmation', with: 'update_password'
          click_button 'Update'
          expect(page).to have_content("User was successfully updated.")
          expect(current_path).to eq user_path(user)
        end
      end

      context 'メールアドレスが未入力' do
        it 'ユーザーの編集が失敗する' do
          visit edit_user_path(user)
          fill_in 'Email', with: ''
          fill_in 'Password', with: 'password'
          fill_in 'Password confirmation', with: 'password'
          click_button 'Update'
          expect(page).to have_content '1 error prohibited this user from being saved:'
          expect(page).to have_content("Email can't be blank")
          expect(current_path).to eq user_path(user)
        end
      end

      context '登録済みのメールアドレスを入力' do
        it 'ユーザーの編集が失敗する' do
          visit edit_user_path(user)
          fill_in 'Email', with: other_user.email
          fill_in 'Password', with: 'password'
          fill_in 'Password confirmation', with: 'password'
          click_button 'Update'
          expect(page).to have_content '1 error prohibited this user from being saved:'
          expect(page).to have_content("Email has already been taken")
          expect(current_path).to eq user_path(user)
        end
      end

      context '他ユーザーの編集ページにアクセス' do
        it 'アクセスが失敗する' do
          visit edit_user_path(other_user)
          expect(page).to have_content("Forbidden access.")
          expect(current_path).to eq user_path(user)
        end
      end
    end

    describe 'マイページ' do
      context 'タスクを作成' do
        it '新規作成されたタスクが表示される' do
          create(:task, title: 'test', status: :doing, user: user)
          visit user_path(user)  
          expect(page).to have_content "You have 1 task."
          expect(page).to have_content "test"
          expect(page).to have_content "doing"
          expect(page).to have_link "Show"
          expect(page).to have_link "Edit"
          expect(page).to have_link "Destroy"
        end
      end
    end
  end
end
```

spec/system/user_sessions_spec.rb
```ruby
require 'rails_helper'

RSpec.describe "UserSessions", type: :system do
  let(:user) { create(:user) }

  describe 'ログイン前' do
    context 'フォームの入力値が正常' do
      it 'ログイン処理が成功する' do
        login(user)
        expect(page).to have_content "Login successful"
        expect(current_path).to eq root_path
      end
    end

    context 'フォームが未入力' do
      it 'ログイン処理が失敗する' do
        visit login_path
        fill_in 'Email', with: ''
        fill_in 'Password', with: ''
        click_button 'Login'
        expect(page).to have_content "Login failed"
        expect(current_path).to eq login_path
      end
    end
  end

  describe 'ログイン後' do
    context 'ログアウトボタンをクリック' do
      it 'ログアウト処理が成功する' do
        login(user)
        click_link 'Logout'
        expect(page).to have_content "Logged out"
        expect(current_path).to eq root_path
      end
    end   
  end
end
```

spec/system/tasks_spec.rb
```ruby
require 'rails_helper'

RSpec.describe "Tasks", type: :system do
  let(:user) { create(:user) }
  let(:task) { create(:task) }

  describe 'ログイン前' do
    describe 'ページの遷移確認' do
      context 'タスクの新規登録ページにアクセス' do
        it '新規登録ページへのアクセスが失敗する' do
          visit new_task_path
          expect(page).to have_content 'Login required'
          expect(current_path).to eq login_path
        end
      end

      context 'タスクの編集ページにアクセス' do
        it 'タスクの編集ページへのアクセスが失敗する' do
          visit edit_task_path(task)
          expect(page).to have_content 'Login required'
          expect(current_path).to eq login_path
        end
      end

      context 'タスクの一覧ページにアクセス' do
        it '全てのユーザーのタスクが表示される' do
          task_list = create_list(:task, 3)
          visit tasks_path
          expect(page).to have_content task_list[0].title
          expect(page).to have_content task_list[1].title
          expect(page).to have_content task_list[2].title
          expect(current_path).to eq tasks_path
        end
      end

      context 'タスクの詳細ページにアクセス' do
        it 'タスクの詳細ページが表示される' do
          visit task_path(task)
          expect(page).to have_content task.title
          expect(current_path).to eq task_path(task)
        end
      end
    end
  end
    
  describe 'ログイン後' do
    before { login(user) }

    describe 'タスクの新規登録' do
      context 'フォームの入力値が正常' do
        it 'タスクの新規作成が成功する' do
          visit new_task_path
          fill_in 'Title', with: 'test_title'
          fill_in 'Content', with: 'test_content'
          select 'doing', from: 'Status'
          fill_in 'Deadline', with: DateTime.new(2023, 3, 22, 10, 30)
          click_button 'Create Task'
          expect(page).to have_content 'Task was successfully created.'
          expect(page).to have_content 'Title: test_title'
          expect(page).to have_content 'Content: test_content'
          expect(page).to have_content 'Status: doing'
          expect(page).to have_content 'Deadline: 2023/3/22 10:30'
          expect(current_path).to eq '/tasks/1'
        end
      end

      context 'タイトルが未入力' do
        it 'タスクの新規作成が失敗する' do
          visit new_task_path
          fill_in 'Title', with: ''
          fill_in 'Content', with: 'test_content'
          select 'doing', from: 'Status'
          fill_in 'Deadline', with: DateTime.new(2023, 3, 22, 10, 30)
          click_button 'Create Task'
          expect(page).to have_content '1 error prohibited this task from being saved:'
          expect(page).to have_content("Title can't be blank")
          expect(current_path).to eq tasks_path            
        end
      end

      context '登録済みのタイトルを入力' do
        it 'タスクの新規作成が失敗する' do
          visit new_task_path
          other_task = create(:task)
          fill_in 'Title', with: other_task.title
          fill_in 'Content', with: 'test_content'
          click_button 'Create Task'
          expect(page).to have_content '1 error prohibited this task from being saved:'
          expect(page).to have_content("Title has already been taken")
          expect(current_path).to eq tasks_path
        end
      end
    end

    describe 'タスクの編集' do
      let!(:task) { create(:task, user: user) }
      let!(:other_task) { create(:task, user: user) }
      before { visit edit_task_path(task) }

      context 'フォームの入力値が正常' do
        it 'タスクの編集に成功する' do
          fill_in 'Title', with: 'title_test'
          select 'done', from: 'Status'
          click_button 'Update Task'
          expect(page).to have_content 'Task was successfully updated.'
          expect(current_path).to eq task_path(task)
        end
      end

      context 'タイトルが未入力' do
        it 'タスクの編集に失敗する' do
          fill_in 'Title', with: ''
          select 'done', from: 'Status'
          click_button 'Update Task'
          expect(page).to have_content '1 error prohibited this task from being saved:'
          expect(page).to have_content("Title can't be blank")
          expect(current_path).to eq task_path(task)
        end
      end

      context '登録済みのタイトルを入力' do
        it 'タスクの編集に失敗する' do
          fill_in 'Title', with: other_task.title
          select 'done', from: 'Status'
          click_button 'Update Task'
          expect(page).to have_content '1 error prohibited this task from being saved:'
          expect(page).to have_content("Title has already been taken")
          expect(current_path).to eq task_path(task)
        end
      end

      context '他ユーザーのタスク編集ページにアクセス' do
        let!(:other_user) { create(:user, email: "other_user@example.com") }
        let!(:other_task) { create(:task, user: other_user) }

        it '編集ページへのアクセスが失敗する' do
          visit edit_task_path(other_user)
          expect(page).to have_content 'Forbidden access.'
          expect(current_path).to eq root_path
        end
      end
    end          

    describe 'タスク削除' do
      let!(:task) { create(:task, user: user) }

      it 'タスクの削除ができること' do
        visit tasks_path
        click_link 'Destroy'
        expect(page.accept_confirm).to eq 'Are you sure?'
        expect(current_path).to eq tasks_path
        expect(page).to have_content("Task was successfully destroyed.")
        expect(page).not_to have_content task.title
      end
    end
  end
end
```

##### create_listで連続するテストデータを作成する。
`FactoryBot.create_list(:task,3)`

`FactoryBot`を省略できる設定をしてるのでこのように書ける。  
`task_list = create_list(:task, 3)`

#####  letとlet!の違い
`let`は遅延読み込みなので、userやtaskが必要になったときだけ作成します。  
一方`let!`は、各テストのブロックが実行される前に作成します。  
今回、編集ページのテストを行うには、予めタスクが必要なため、`let!`で作成しておきます。

### [Rspec](http://d.hatena.ne.jp/keyword/Rspec)の実行

```
% bundle exec rspec spec/system/tasks_spec.rb
```

### [RSpec](http://d.hatena.ne.jp/keyword/RSpec)の結果出力を見易く表示する。

以下のオプションを追記する。
.rspec
```ruby
--require spec_helper
--format documentation #追記
```

