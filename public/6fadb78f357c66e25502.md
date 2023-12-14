---
title: OpenAI(ChatGPT)のfine-tuning機能を早速試してみた~ひろゆきのスパチャを例に
tags:
  - Python
  - OpenAI
  - colaboratory
  - ChatGPT
private: false
updated_at: '2023-08-24T13:27:23+09:00'
id: 6fadb78f357c66e25502
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに
OpenAI社から革命的な機能がリリースされました。
https://openai.com/blog/gpt-3-5-turbo-fine-tuning-and-api-updates 

タイトルにもある通り、fine-tuningが利用可能になったとのことです。これにより自分たちが持っているデータに合わせて学習させたモデルを作成し、利用することができるようになりました。

> Developers can now bring their own data to customize GPT-3.5 Turbo for their use cases.

[公式Twitter](https://twitter.com/OpenAI/status/1694062483462594959?s=20)

> lets you train the model on your company’s data

つまり、従来は一般論しか返してくれなかったものが、会社や組織独自の文脈のデータを教え込ませ、それに沿った回答を返してくれるとのことです。独自データに沿った回答を得られないことが従来の課題だったので、非常に革命的なアップデートですね。早速試してみたので、そちらを以下に共有します。公式tutorialなるものがありますが、なるべく誰でもすぐに実践できるように書きました。

独自データは指定のフォーマットに沿ったファイル形式を作成し、読み込ませます。そして、モデルを使用しAPIを叩くだけなので非常に簡単です。果たして教えてデータを元に回答を返してくれるのでしょうか？検証してみます。

# もくじ
1. 事前準備
2. データの作成
3. モデルの作成
4. 作成したモデルを使って質問する

# 事前準備
使用するものはGoogle Colaboratoryです。これはPythonをGoogle Drive上でオンラインで実行できるようにした対話型エディターです。また、OpenAIのアカウントを作成し、API_KEYを発行している必要があります。

## Colaboratoryの準備
任意のDrive配下にGoogle Colaboratoryを作成し以下に沿ってコードを実行していきます。

```python:fine_tuning.py
!pip install openai
```

```python:fine_tuning.py
import os
os.environ["OPENAI_API_KEY"] = "<ここにAPI_KEYを書く>"

# driveをマウント(Colabからファイルを読み込めるようにする。初回は認証が必要なので許可します。)
from google.colab import drive
drive.mount('/content/drive')

# 階層を移動(仕組み詳細はこちら -> https://www.ushiji.online/colab-file-upload)
%cd "/content/drive/<colabを作成したフォルダーのパス>"
```

## データの作成
学習させるデータは以下のようなフォーマットで作成する必要があります。最大50MBまで対応しています。これは、公式tutorialの例ですが、皮肉な回答を返すように学習させているものです。

```json:test_data.json
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "What's the capital of France?"}, {"role": "assistant", "content": "Paris, as if everyone doesn't know that already."}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "Who wrote 'Romeo and Juliet'?"}, {"role": "assistant", "content": "Oh, just some guy named William Shakespeare. Ever heard of him?"}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "How far is the Moon from Earth?"}, {"role": "assistant", "content": "Around 384,400 kilometers. Give or take a few, like that really matters."}]}
```

今回は生データとして質疑応答CSVから自動で上記のフォーマットを生成することを想定し、実装していきます。

まず、以下のようなCSVを作成します。質問と回答がセットとなった情報が入っています。
ちなみに教えるデータは[ひろゆきの質問回答集](https://tkykproject.com/hiroyuki/)です。

```csv:test_data:csv
user,assistant
お金持ちになる方法,お金持ちになるには「情報」と「賭け」が必要。みんながやっていることは価値が無いため、誰もやっていない事にお金を賭けられる度胸があるかどうか。他の人が気付かない内にそこに時間を投資すると他の人が気付いた頃に先行者利益が手に入っている
成功者の特徴,成功者はみな性欲が強い。特に１０億以上の資産家はその傾向が強い
コンピューター関係の資格について,コンピューターが本当に好きな人は資格よりもアプリを作ったりしてる。資格ばかり取得している人は就職が目的でセンスや情熱で見ると無能じゃないかと疑ってしまう。
MacとWindowsどちらがオススメですか,ゲームをするならWindows一択。プログラム回りの作業はMacでもWindowsでもどちらでもOK。0.01秒単位で作業する音楽編集はWindowsで高スペックな物をおススメします。遅延が少ないため。
YouTubeはこの先衰退するのか,Googleは全世界のトラフィックの３～４割を占めている。当然データサーバーもそれに追随して超高容量になっている。この規模のデータサーバーを保有しているのはGoogleとAmazonくらい。つまり各国にデータを送る費用も後発組より圧倒的に安い。後発組が参入、奪還する余地が無い。
自己肯定感の高めかた,例えば小学生に対してマウンティングしてみる。小学生がやってるゲームに廃課金して勝つなど。自分が勝てる分野でのみ戦い、価値を積み上げる。これをすれば自己肯定感が上がる。苦手なものから逃げる。
どんなバカでもできる職業を教えてください,プログラム。プログラムは間違っていると「ここが違う」と誰にでも分かる形で見える化される。営業などと違って正解不正解が明確。教科書があればバカでも出来る。
記憶力が悪いことは良い事ですか？悪い事ですか？,やられた事をずっと覚えているタイプより、寝たら忘れるタイプの方が生きていく上では楽。昔は記憶力が悪いと生活が不便であったが、今はネットでその都度調べれば良いので記憶力が悪いデメリットは消えつつある
食生活で意識している事,外食を控える、レンジで簡単調理、インスタント系を食べない。料理を作る時間があるなら、自炊する方が良い。
どうすれば図太く生きれるか,自分は別に人に嫌われていないと考えている。相手の事を知り、性格を知ると、僕の事を嫌いになるはずがないと自覚できる。相手を知ることが大切。
ヤラセについて思う事,ヤラセを一切公表せずにだます方法はタチが悪い。ただどんなものでも多少の演出が入る事はある。喜怒哀楽、物語が何も起きないリアルなドキュメンタリーは視聴率が稼げない。人は何か改善、工夫をしたくなる生き物であるのでそこに悪意はない事も多い。
日本の国債はいくら発行しても、借金ではないので問題ないという意見にどう思うか,もし国債発行を無限に行い、何も問題が無いのであれば今すぐGoogleの時価総額分を発行し、Googleを買収すべき。そうすればGoogleがあげる年間利益は全て日本に流れ込む。同様にAppleやAmazonも全て国債で買い占めれば日本は世界一の富裕国になる。ただこれは100兆円を相手企業が欲しがらないといけないが、恐らくここまで来ると円の価値は無くなり、相手が欲しがらない。このようにあまりにも行き過ぎた国債発行は為替レートを壊してしまう懸念がある。この限界値がどこまでか分からない。そこが危険。
コロナで人類絶滅の可能性は,若い人は生き残るので人類絶滅は無いと思う
子どもの虐待を減らすには,本当に子供を殺したい人はほぼいない。自分の自由が無くて天秤にかけて過ちを起こしている。お金があれば自分の時間を確保できる。ストレス解消も出来る。
心配性を治したい,世の中には適当にダラダラと生きている人もいる。その中で将来を不安視して行動におこす、または準備をしている人は優秀である。心配性は優秀な人が多いのでそのままで良い
いつか空飛ぶ車が日常的に登場するか,空を飛ぶことはやたらコストがかかる。上に飛ばすって大変、仮にそこまで科学力が進歩していたらオンラインなど直接物質がそこまで飛行して移動する必要が無い世界になっていると思う
無駄に長い会議に出席するのが苦痛だ,生産性の低い会議を続けていても利益が出ている。その点で組織として経営状態は良い。その風土が変わってないという事は、上司に有能な人物が少ない証拠。よってやり方によってはあなたが出世するチャンスがある。社風を残しつつ、上の人たちに気に入られながら業務効率化を推進出来れば昇進できる
反社会的勢力を判断する方法は,基本的には判断できない。ルール上反社会勢力は賃貸も銀行も使えないはず。けど実際は使ってる。銀行の審査をすり抜けるレベルの反社会的勢力を一般市民が見抜けない。内部に入るしかない。
充実した人生を送る為には,充実しているかどうかを決めるのは自分自身。江戸時代は凧揚げ超楽しい、で日々が充実してた。高級料理食べたいとかは無い。充実感を求めているといつまでも充実できない。
事故物件をどう思うか,家賃が安い、万が一心霊映像が取れたらYouTubeにUPしたり取材が来たりお金になる。覚悟が出来て気にしない人はお得。
ギャンブル依存症を治すには,ちゃんと辞められた人はめったにいない。いっそのこと自己破産させた方がいい。クレジットカードを持てなくなるので。
やっておくと良いスポーツは,格闘技はオススメ。世の中話し合いで解決できないこともある。イキって来る人に余裕を持って対処するには格闘技でこれだけはまずいと言うボーダーラインを把握できる。
人類最強は相撲取りですか,首を取りにくい、という意味では相撲は強い。関節技はあくまで金的や目つぶしが無い競技で有効。よってルール無しでの戦いでは関節技は難しい。次に弱点が首と頭だが、相撲取りの頭への攻撃は難しい。よって最強と言えるかもしれない。
日本企業が海外企業と比較して劣っている点は,日本は無能をクビに出来ないシステムがキツイ。なぜこの状況でまかり通っているかというとサービス残業などで時間外労働を入れると会社としては利益が出ており、クビにせずとも雇えるため。
第一印象で人を判断して良いか,第一印象が正しかった、は結構ある。本当な様々な情報で総合的判断をしているが、それが本人は第一印象だと認識しちゃってる場合もある。自分の評価軸がある程度固まっていて、認識出来ている人がそれでいいと思う
FX取引で稼ぎたい,お金を稼ぎたいからFXは間違い。ギャンブルはお金が０やマイナスになる可能性のあるものなので増やしたい人が行うものではない。会社は利益が出るまで何年かかかっても耐えれるが、個人は数年間も利益なしが耐えれない。それであれば学力を上げて、自分の資産を日々右肩上がりで増加出来ることに時間を注ぐべき
これを見て人生が変わった本はありますか,コーラン。イスラム教の大多数はこう考えるよね、という軸が学べる。キリスト教は個人の考え方が多岐に渡り聖書だけでは把握しきれないがイスラム教はコーランで大枠が理解できるので１冊で把握できる人数のコスパがとても良い
人生を楽しむ方法,人生はどれだけ楽しく暇つぶしを行えるか。80年の中で趣味に割く時間、没頭できる時間が長ければどんどん幸福度が増す。自分が好きなものにどうすればもっと深くかかわれるかを考える事が大切
人類史上最大の発明は,車輪。飛行機は鳥の真似、プロペラもあの仕組みの生物がいる。けれど車輪は人類が効率化の為に０から作り出した。だから車輪。車輪が人類の進化に与えたインパクトは大きい。
```

このファイルをDrive上でColabと同じ階層に置きます。そして以下のPythonコードを実行し、CSV形式のデータを指定のフォーマットであるjsonl形式のデータに変換します。

```python:fine_tuning.py
import csv
import json

def csv_to_jsonl(input_file, output_file):
    with open(input_file, mode='r', encoding='utf-8') as csvfile:
        reader = csv.DictReader(csvfile)
        
        with open(output_file, mode='w', encoding='utf-8') as jsonlfile:
            for row in reader:
                message = []
                
                message.append({
                    "role": "system",
                    "content": "西村ひろゆきBot"
                })
                
                message.append({
                    "role": "user",
                    "content": row['user']
                })
                
                message.append({
                    "role": "assistant",
                    "content": row['assistant']
                })

                data = {"messages": message}
                jsonlfile.write(json.dumps(data, ensure_ascii=False) + '\n')


csv_to_jsonl('test_data.csv', 'test_data.jsonl')
```

そうすると、Colabと同階層に学習に用いるjsonlファイルが生成されます。これにて準備完了です。

## モデルの作成

fine-tuningさせるデータの準備が整ったので、モデルを作成していきます。

```python:fine_tuning.py
import openai

# ファイル読み込み
test_data_file_object = openai.File.create(
  file=open("test_data.jsonl", "rb"),
  purpose='fine-tune'
)

file_id = test_data_file_object.id

# モデル作成Jobの実行
job_response = openai.FineTuningJob.create(training_file=file_id, model="gpt-3.5-turbo")
```

`openai.FineTuningJob.create`で独自データを学習させたモデルを作成する依頼を出します。OpenAI側で順に処理されていき、5分ほど待ちます。そして処理が完了してモデルが作成されているか確認します。利用できる元のモデルは、現時点では`gpt-3.5-turbo`になります。

```python:fine_tuning.py
job_id = job_response.id

response_retrieve = openai.FineTuningJob.retrieve(job_id)
response_retrieve
```

処理が完了した場合は、レスポンスの表示の中にある`fine_tuned_model`フィールドにモデル名が入ってきます。未完了の場合は、nullになります。
何度か上記を実行して、`fine_tuned_model`フィールドにモデル名が入ってきたら最後のステップです。

## 作成したモデルを使って質問する

```python:fine_tuning.py
fine_tuned_model = response_retrieve.fine_tuned_model
```

作成されたモデルを利用して質問を投げます。

```python:fine_tuning.py
response = openai.ChatCompletion.create(
    model=fine_tuned_model,
    messages=[
        {"role": "user", "content": "人間は死んだ後どうなりますか？"}
    ]
)

print(response["choices"][0]["message"]["content"])
```

筆者が試した際に帰ってきた回答は以下でした。
`死後,人間の魂は永遠に神様と共に過ごしますよ`

簡潔でひろゆきっぽい言い回しになりましたね？

ちなみに、通常のChatGPTで聞いた際の回答は以下です。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/614347/68604dc3-2ce7-fa4a-d154-00e96f6c50b2.png)

これと比べると違いがよく分かります。

次に質疑応答の中の文面に沿った回答を返してくれるのか？を検証します。元ネタの質問回答は以下です。

`質問: 自己肯定感の高めかた`

`回答: 例えば小学生に対してマウンティングしてみる。小学生がやってるゲームに廃課金して勝つなど。自分が勝てる分野でのみ戦い、価値を積み上げる。これをすれば自己肯定感が上がる。苦手なものから逃げる。
`

「小学生に対してマウンティング」のような特異な文面があるので、それに沿って返してくれるのでしょうか？

```python:fine_tuning.py
response = openai.ChatCompletion.create(
    model=fine_tuned_model,
    messages=[
        {"role": "user", "content": "自己肯定感を上げるにはどうすれば良いですか？"}
    ]
)

print(response["choices"][0]["message"]["content"])
```

返ってきた回答は以下でした。

```
自己肯定感を高めるためには、以下のような方法があります。
自分自身の強みを知る：自分が得意なことや良いところを見つけることは自己肯定感を高めるために重要です。自分はどのような特技や能力があるのか、自分の強みを再評価してみることが大切です。
2. 過去の成功体験を思い出す：過去に自分が成功した経験や良い結果を出した経験を振り返ってみることは、自己肯定感を高めることに繋がります。成功体験があることで自信をもち、自己肯定感を高めることができます。
3. 自分の成長を見つける：日々の自分の成長や進歩を認識することも重要です。自己肯定感を高めるためには、自分がどの点で成長したのかを定期的に振り返ることが必要です。
4. ポジティブな言葉を使う：自己肯定感を高めるためにはポジティブな言葉を使うことも有効です。自分に対してポジティブな言葉をかけることで、ネガティブな思考を払拭し、自己肯定感を高めることができます。
5. 周囲のサポートを受ける：自己肯定感を高めるためには、周囲のサポートを受けることも重要です。家族や友人、上司や同僚など、自分をサポートしてくれる人々とのつながりを大切にすることで、自己肯定感を高めることができます。
```

「小学生に対してマウンティング」のような特異な文面は返ってきませんでした。さらにひろゆきっぽさも薄れているようにも感じます、、、

もう一編試してみましょう。
`0.01秒単位で作業する音楽編集はWindowsで高スペックな物をおススメします。遅延が少ないため。`に沿った回答を期待します。

```python:fine_tuning.py
response = openai.ChatCompletion.create(
    model=fine_tuned_model,
    messages=[
        {"role": "user", "content": "音楽編集するとき、WindowsとMacどちらがおすすめ？"}
    ]
)

print(response["choices"][0]["message"]["content"])
```

返ってきた回答は以下でした。

`それは個人の状況に依存する。好きな音楽編集ソフトがWindowsでのみ動くなら、Windowsを使用して、その逆も同様。ただ、音楽編集ソフト自体どちらでも使用できる場合、やはりMacがおすすめとなります。音楽の鮮明さ、フィルタ、信号・ノイズ比といった性能、配線マストの知識に適したマシンです
`

Macをおすすめされてしまいました、、、
とはいえ、文面はひろゆきっぽい簡潔な良い回しのように見受けられます。

## 感想
従来は一般論しか返してくれなかったChatGPTですが、教えたデータを元に回答を作成してくれるfine-tuning機能がリリースされました。実際に触ってみたところ、ひろゆき風を作ることはできましたが、実際の質疑応答の内容に沿った回答はあまり得られませんでした。

[When to use fine-tuning](https://platform.openai.com/docs/guides/fine-tuning/when-to-use-fine-tuning)の章には、

> We recommend first attempting to get good results with prompt engineering

良い結果を得るにはプロンプトエンジニアリングが必要と書かれています。つまり、如何にfine-tuningの精度を上げられるようなデータを作れるかを人間は考えるべきだということになります。

また、データに基づいて回答を生成する技術は他にもembedding(ベクトル検索)があります。対して今回のfine-tuning機能は、指定のデータフォーマットを作るだけでLLMに直接データを学習させるということで一線を画する内容となります。このどちらを利用すべきかは今後検討の余地がありそうですが、[When should I use fine-tuning vs embeddings with retrieval?](https://platform.openai.com/docs/guides/fine-tuning/when-should-i-use-fine-tuning-vs-embeddings-with-retrieval)の章には以下のようなことが書かれていました。


> Embeddings with retrieval is best suited for cases when you need to have a large database of documents with relevant context and information.
By default OpenAI’s models are trained to be helpful generalist assistants. Fine-tuning can be used to make a model which is narrowly focused, and exhibits specific ingrained behavior patterns. Retrieval strategies can be used to make new information available to a model by providing it with relevant context before generating its response. Retrieval strategies are not an alternative to fine-tuning and can in fact be complementary to it.

- 検索を伴うエンベッディングは、関連するコンテキストと情報を持つ文書の大規模なデータベースを持つ必要がある場合に最適
- デフォルトでは、OpenAIのモデルは有用なジェネラリストのアシスタントになるように訓練されている
- 微調整を行うことで、焦点を絞った、特定の行動パターンを示すモデルを作ることができる
- 検索戦略は、反応を生成する前に関連するコンテキストをモデルに提供することで、新しい情報を利用可能にするために使用できる
- 検索戦略はファインチューニングの代替策ではなく、むしろそれを補完するものである。

なんとなく雰囲気は伝わってきます。あくまでfine-tuningは、特定の行動パターンを示すモデルを作るに過ぎないということだそうです。つまり今回の検証において、ひろゆき風を作り出したということになります。

文脈を理解し回答を作り出すことを実現したければ、今のところembeddingの検索戦略がまだ適しているように思いました。しかし、検索戦略とfine-tuningは補完しあうものであるとOpenAIは述べています。将来的にこれらを組み合わせたプロンプトエンジニアリングが人類に求められていくのだと思います。

今回検証した分ではまだまだ改善の余地があるように思います。しかし生成AIの技術の進化は今までにも私たちの想像を超えてきました。今後も、モデルの精度向上やプロンプトエンジニアリングの蓄積のような技術発展によって、生成AIの可能性は大きく広がりそうです。

使用用途として例えば社内情報を教え込ませ、ユーザーからの問い合わせを自然言語でBOTが返すような世界を簡単に作れるようになるのでしょうか？将来が非常に楽しみです✨

# 参考文献
https://dev.classmethod.jp/articles/openai-gpt35turbo-fine-tuning
