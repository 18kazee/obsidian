
```ruby
# Rails実践カリキュラム基礎編の課題内容の学習時間をグループ分けして出力してください。
# curriculum_dataは["課題項目", "学習時間"]で配列になっています。
curriculum_data = [
  ["課題1 アプリの基本設定を行う", 2],
  ["課題2 全ページにヘッダー/フッターを設置", 6],
  ["課題3 Gemを使ってみよう (Bootstrap)", 2],
  ["課題4 sorceryを使用して、ユーザー機能を作成しよう", 16],
  ["Sorcery課題", 4],
  ["課題5 i18nによる日本語化対応", 4],
  ["課題6 フラッシュメッセージの設定", 6],
  ["課題7 デコレーターの導入", 4],
  ["課題8 掲示板の一覧機能の作成", 8],
  ["課題9 掲示板作成機能", 8],
  ["課題10 フォーム入力時エラー情報を個別表示", 4],
  ["BugFix課題", 3],
  ["課題11 掲示板の画像アップロード機能", 12],
  ["課題12 掲示板詳細画面の追加/コメント機能の実装", 16],
  ["課題13 タイトルを動的に出力する", 4],
  ["課題14 掲示板の編集、削除機能の実装", 8],
  ["課題15 ブックマーク機能の追加", 12],
  ["課題16 ブックマークボタンのajax化", 8],
  ["Like課題", 4],
  ["課題17 コメント投稿、削除、編集機能のajax化" ,12],
  ["課題18 掲示板のページネーション", 8],
  ["課題19 掲示板の検索機能を実装", 8],
  ["課題20 プロフィール編集機能の実装", 4],
  ["Profile課題", 4],
  ["課題21 パスワードリセット機能の実装", 16],
  ["課題22 [管理画面] 管理画面へのログイン機能、管理画面トップページの作成", 12],
  ["課題23 [管理画面]掲示板/ユーザのCRUD機能の作成", 8],
  ["Admin課題", 4]
]

sorted_curriculum_data = curriculum_data.sort_by { |data| -data[1] }
grouped_curriculum_data = sorted_curriculum_data.group_by { |data| data[1] }

output = grouped_curriculum_data.map do |hour, data_array|
  {
    hour: hour,
    count: data_array.size,
    contents: data_array.map { |data| data[0] }
  }
end

curriculum_data_contents_order = curriculum_data.map { |data| data[0] }

output.each do |item|
  item[:contents].sort_by! { |content| curriculum_data_contents_order.index(content) }
end

curriculum_data.clear.concat(output)

# 出力は下記1行で行うものとする
print curriculum_data
```


- `sorted_curriculum_data = curriculum_data.sort_by { |data| -data[1] }`
`curriculum_data` の2番目の要素である時間でソートし、ソート順を逆にして `sorted_curriculum_data` に代入します。

- `grouped_curriculum_data = sorted_curriculum_data.group_by { |data| data[1] }`
`sorted_curriculum_data` を時間（2番目の要素）でグループ化し、`grouped_curriculum_data` に代入します。

- `output = grouped_curriculum_data.map do |hour, data_array|   {     hour: hour,     count: data_array.size,     contents: data_array.map { |data| data[0] }   } end`
`grouped_curriculum_data` の各グループに対して、以下の形式のハッシュを `output` 配列に追加します：
`{   hour: <時間>,   count: <その時間のコンテンツの数>,   contents: [<その時間のコンテンツのリスト>] }`

- `curriculum_data_contents_order = curriculum_data.map { |data| data[0] }`
`curriculum_data` の各要素の1番目の要素を抽出して `curriculum_data_contents_order` 配列に代入します。これにより、後の処理で `curriculum_data` 内の要素の順序に従ってコンテンツが並び替えられます。

- `output.each do |item|   item[:contents].sort_by! { |content| curriculum_data_contents_order.index(content) } end`
`output` 内の各要素の `contents` 配列を、`curriculum_data_contents_order` に従って並び替えます。

- `curriculum_data.clear.concat(output) print curriculum_data`
最後に、`curriculum_data` を `output` で上書きし、標準出力に出力します。