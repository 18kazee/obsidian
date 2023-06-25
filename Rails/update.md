
以下のコードの問題点を挙げるとどうなるか。

答えは、updateが誰のでも行えるところに気づくべき。
`@profile = current_user.profile`  のようにして自分のprofileのみを更新するべき

```ruby
# ※ ユーザーモデルです
class User < ApplicationRecord
  has_one :profile
end

# ※ 自身のプロフィールを編集するためのコントローラーです
class ProfileController < ApplicationController
  # === 省略 ===
  def update
    @profile = Profile.find(params[:id])
    if @profile.update(profile_params)
      redirect_to xxx_path
    else
      render :edit
    end
  end

  private

  def profile_params
    params.require(:profile).permit(:name, :age)
  end
end
```
