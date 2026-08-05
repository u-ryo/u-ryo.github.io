---
layout: post
title: "learned on ruby and vue, angularjs"
date: "2020-03-15 21:15"
author: 'u-ryo'
categories: [ruby, vue, angularjs]
comments: true
published: true
---
お仕事でRuby on RailsとVueをやっております。
これまではずっとJavaやAngularな人生だったので、Duck Typingと格闘中です。
新規機能開発で様々な山を超えてきた(知見を得た)ので、忘れないうちに。

- 破壊的method(Bang Method)って、その響きから何かあんまり良くないのかと無意識に思っていたのですけれども、調べてみると特に忌避すべきというものは見つかりませんでした。内部logicを考えると中で変数を付け替えていると思われますが、bangしないと両方分memoryに持ってなきゃならないから?→そうしたらrubocopに、`result[:key]=... if ...`と書けと言われました。ナルホドです!
- 手元では`yarn test:vue`通るのにCircleCIでは`7 fails`と言われて通らないので悩みました。どう目を凝らしてもerrorらしき出力はなく。よく見るとCircleCIの最後に`Too long with no output (exceeded 10m0s): context deadline exceeded`とあり、かかった時間が10:57とか。えー!?そんなにかかっているの?? 手元で`time`で計測するも、1分ちょっと。CircleCI再実行して観察すると、test自体は1分程で終わり、その後ずっとだんまりで、10分程してTimeoutします。何だろう? 手元では再現しないのでとにかく厄介です。考えてみたら、チェックじゃなくて「1)」とか「2)」になっているところが失敗したところなんですね。確かに7)までありますわ。でも、そこがどう間違っているのかが全然出力されないので皆目わかりません。一箇所、`describe('...', async () => {`ってなっていたから、これか! と消して勇躍試してみたものの、症状変わらず。`mount`を`shallow`にしてみたり削れるだけの`async`/`await`を削っても。何故か`describe`の囲みを解くとtestが失敗します。期待してるobjectのpropsがないと。でももう一つ別のpropsはある。もぅわけわかんない!ですよ。CircleCIにsshで入ったりして色々と調べてみると、特定の2 filesが先にあると失敗する様子。何それ。どちらかがない、もしくはどちらかより先に実行されれば上手く行く。もっと詳しく見てみると、具体的には、`shallow`した直後の`wrapper.html()`の中身が全然違うという。どうして? そんなとこどうにもなんないじゃん。...あぁ、ぃゃ、test対象Vue Classできちんとcomponentを定義すると(e.g.`components: { 'el-button': Button, 'el-dialog': Dialog }`)warningが消えてcomponentとして見えるようになることがありますが(特にElementUIは`el-...`とclass nameが合わないので...(- -;)、そういうわけでもなく。何せ2 filesが組み合わさるとtestにコケるというのが謎です。試しにダメにするspec fileを一つ消してから全体的に`npm run test:vue`してみると、またコケるので、その2 filesはat leastであってもっと他にも組み合わせの悪いのがあるということです。i18nかなぁ? 確かに一方はenで他方はjaでしたが、大して使ってないのでcomment outしても変わらないですし... localでは全体的に通しても通っている、CircleCI上でも単体や相性の悪いものの後でなければ動く、でも全体的に通すとtestに失敗する、というのはとても悔しく、厄介です。結局心折れて、componentが見つからなければtest codeをthroughするようにしました。端から`describe.skip(...`よりはマシでしょう。それにしても何でやねん。大したtestしてるわけでもないclasses(methodがcallされたかどうかをassertしてる程度)に阻まれて、ちゃんと作ったspecのtest(条件がどうならどういう表示状態、`input`要素をclickしたらどういう文言のdialogが出て、等)を実行できないというのは何とも残念でなりません。
- Vueのtest、ムズカシイです。child componentの扱い、ですね。何気なく使っているであろう`el-table`とか`el-dialog`、そのままだとそんなcomponent知らないよとwarningが出まくります(`[Vue warn]: Unknown custom element: <el-table-column> - did you register the component correctly? For recursive components, make sure to provide the "name" option.`)。`wrapper`からはHTML Tagとして扱えるので最後はそうするのですけど、出来ればちゃんとcomponentとして扱いたいところです(でないとtable等でdataをloopさせて表示を作るcodeだった時のdata部分がそっくりないのでtest出来ない)。spec classで`shallow`(or`mount`)する時に`stubs:`するのかと思ってた(前それで頑張った気がした)んですけどそうではなく、spec対象元classでちゃんと定義されていれば?、否、ElementUIなんかは丸っと読み込まれているのでいいのかなぁ?ともあれ、spec classで`localVue.use(Table)`とした`localVue`を`shallow`(or`mount`)時に読み込んでやる必要がありました。でそのcomponentを`find`して`.props()`して中身を`expect`していくという流れですね。最初は`console.log(wrapper.html())`で何が捕れるのかよく観察すると。でも上記のように、何かが変わるとすぐ取れるcomponentが変わるようなので、怖いです。その辺りの機微がよくわからないので...
- Vue testにおけるtickの待ち方。`it(..., async () => { ... await Vue.nextTick(); ...});` 2 ticks待たないと値が変わらないことも。
- `wrapper.findAll('.cell')`で取って来た値はElement Objectなので、見た目上は空でも`expect(cell).to.be.empty`とは出来ません。`expect(cell.isEmpty()).to.be.true`
- Vue componentって、初期値`props`と`data`は分けねばならない? 当初、`data`に初期値を渡そうとしてどうしても出来ず、途方に暮れました。`data`の初期値は`data`部に書いた最初に返す値でそれは何でもよく、tagの方で渡す値で上書きされるのかと思ったら違うんですね。では`data`ではなく`props`を使うのかと思うとそうでもなく、`props`を後から変更しようとすると怒られますし。Stackoverflowにあった解決策は、初期値`props`と`data`を使い分けること。無駄な変数増えて何だかなぁとは思いますれど、そういうもの?
- `slot-scope`って、`="scope"`ではなく`={row}`とやればいいんですね。本で知りました。
- `...mapGetters`がなかなか効かなくて苦労しました。`computed:`に入れる、storeのmodule構造に合わせて`module(file)名/method名`にする、とやったんですが、`undefined`って言われて。`store`経由で`this.store.value_name`とかってやると取れるのに。でも[State を直接参照せず、getter 経由で参照することを強く推奨する](https://uncle-javascript.com/vuex-getters)というので頑張りました。一回は諦めたんですけど、その後retryしたらうまくいくように。結局どうしてうまくようになったのか、決め手が何だったのかよく分からずじまいです。
- 確認dialogを出すというんですが、非同期のJavascript/Vueにあってはこれが意外とムズカシイです。確認dialogというからには処理を止めなければなりません。よくやるように「flagで表示を制御」ではmain処理止まらないんですよね。そもそもsleepだとて簡単には行かないですよねJavascriptで。とても苦労しました。`async`/`await`を導入してやっと止めました。要は「dialogを作っているところを`new Promise`でくるんで`return`、それを`await`、しているfunctionを`async`」ですかね。
- dialogって自分で`$destory()`出来るんですね。生成と削除は外からやらないとダメかな、と思っていた(ので`$emit`して親で受けて`dialog.$destory()`してた)んですけど、実際生成は外から`$mount()`呼ばないとならないですけど(`created:`や`mounted:`はcallbackですよね)、「ボタン押したら消滅」で自己`$destroy()`は出来るんですねやってみたら。いちいち面倒くさいけどやってみないとわからんもんですね。
- dialogを出す必要に迫られたんですけど、よくあるようにdialogを予めDOMに仕込んでflagでappear/disposeとするには、2階層上のVue componentに書かないと、loopで大量生産されてしまうことが判明。Vue component間は親との対話だったら`$emit`と、戻しは何だ??を使えば出来ますが、2階層上は面倒です。情報渡すだけならまだしも、その後の処理を孫でやるならdialogの結果を2階層戻してもらわないとならないですし、じゃぁ処理もgrandparentに渡す? 処理に必要な情報も渡さないとだし、grandparentで知らなくてもいいことを知らなきゃいけないなんて。ということで、何かないかなーと探すと、揮発性dialogというものが([揮発性の高いコンポーネントを作る話](https://speakerdeck.com/uhck/hui-fa-xing-falsegao-ikonponentowozuo-ruhua))。探すと似たようなことしてる人割と多く。これを理解して、そのままではなく2 filesで済むようにして自分のにapplyするのも頭を使いました。これ結構いいような。Vueのdialogも一般的にこれでいいような。初からDOMについているのではなくって。
- 既述のように、`await`もうまく行ってよし、じゃぁtestを書くぞ、と試したら、いきなりわけのわからないerrorに見舞われてそもそもtestが動かなくなってしまいました。なんでだろー?! 自分以前では確かに動いていたので、自分のせいです。でもどのfileが悪いとかも出て来ないのでまるで取っ掛かりがなく、途方に暮れました。error messageで検索すると、どうやらなんと`async`/`await`を使った場合に出る`bable`/`polyfill`系のerrorだとか。えー!! 折角苦労して`async`/`await`でdialog止めたのに! 代替手段なんて思い付かないよー! ということで、何とか突破すべく頑張りました。`babel`の線で色々と調べてみると、こうしたら直った、ああしたら、とか種々書いてあるんですね。新たなgem install以外は全部試しましたけど、うまく行かず。でも[この記事](https://stackoverflow.com/questions/33527653/babel-6-regeneratorruntime-is-not-defined/)を参考にvueの`setup.js`冒頭に`require("babel-polifill");`と書いたら!! 動いてくれました!! 省みれば僅か1行の追加ですけど、この1行には数時間もの汗と涙が詰まっているのです。
- APIの試験をするのに、`curl`やPostmanを使うのはよくありますけど、cookieが何か長くてcopyが面倒だなーと思ったので、Chrome Developer toolのConsoleにて`fetch`でやる手法を。`fetch(｀/apis/endpoint/＄｛id｝｀,{method:method,credentials:'include',headers:{'X-CSRF-Token':token}}).then((response)=>console.log(response))`とすればBrowserの持っている認証情報をそのまま使ってくれます。`credentials:'include'`が肝です。どうせ使い捨てcommandですから。
- Railsは自動的にCSRF対策がなされている、ということで、何してるんだろーと思いきや、session毎に予測困難な`X-CSRF-Token`が付くというものでした。request毎に違うものでなくてもいいのか? いいのか...予測困難なら...
- Railsの日付、Timezone気を付けないといけないのですけど(AWSはUTCなので)、`Time.current`や`Date.current`のように`.current` methodを使えばTimezoneを意識してくれる模様。`.now`より`.current`がということでやはり「ナウい」は死語ということですか。
- Ruby、共通処理は継承関係作って親クラスに、と思っていたんですけどそれってJava的発想? OOP的というか。RubyではむしろMixin? MixinってAOPかと思ってたんですけど。Duck Typingだからそもそもclass的には無関係でいいんですね。Java屋さんとしては何だかなぁと。
- Rubyのstream(`map`や`select`や`reject`や`each`等)、もうfoolproofとして`lazy...force`を付けるのがいいんじゃないか、と思ったんですけど、そうでもないんですねこれ!! ってかそもそもどうして`lazy`がdefaultじゃないんだろう? ってJava屋さんとしては思ってたんですけど、[銀の弾丸ではない](https://bugs.ruby-lang.org/issues/6183#note-2)という話があって衝撃でした何を今更ですけど(古い話のようで今は[多少performance上がった](http://blade.nagaokaut.ac.jp/cgi-bin/scat.rb/ruby/ruby-core/77130)そう)。というわけで、`lazy`は「省メモリ」か「後でかなりの数`take`することがわかっている場合」でないと効果なさそうなんですね。[元ネタ](https://stackoverflow.com/questions/47653131/rubys-lazy-less-performant-than-allocating-huge-lists)
- 最近何か安易に大量のlogをslackに投げていたので、そうじゃないでしょ基本的にはlog fileでしょ、それもdefaultのfileでいいよね、ということで、`Logger.new(...)`ってやったんですけどどうして毎回file nameを指定しなきゃいけないのか、しかもfile名指定して`new`すると都度file作りやがる、のでappend modeでopenして、とかってやらないとならないの?! なんか面倒臭くね? と思ってたら、`Rails.logger`でいいんですね。なぁんだ。`@logger ||= Rails.logger`
- `controller`で最終的?に全てのerrorを拾うには`rescue_from error_class名, with: :method名`、事前validationを定義するには`before_action :method名`
- 決まった名前をclass methodにするか定数にするか、は、定数の方が良いようです。ref. [Class method vs constant in Ruby/Rails](https://stackoverflow.com/questions/15903835/class-method-vs-constant-in-ruby-rails)
- Rubyで「objectの同一性」とは、Javaと違って`==`、`eql?`、`hash`の3 methodあるんですね。うち`==`はArrayで、残る2つはSetで使われる模様。Arrayでは比較通るのにSetでは違うと判定され悩みました。Setではobject hashを取っているからなんですね。でも結局、`.to_json`すれば同一性methods定義しなくても行けました。何だかなぁですけどまぁunit testなのでいっかー。
- すっかり忘れてましたけど`routes`で`resources`を使えばendpoint定義をまとめられますね。path parameterも積極的に使うべきでしょう(その存在を強制できるので)。parameter nameは基本`id`ですけど変えられるんですね。
- ActiveRecordで、MySQLみたいに「あれば取得、なければ作る(recordも入れる)」なんてあるかなーと思ったら`.find_or_create_by`がありました。更にforeign key制約をつけて、その制約に反したらerrorにして欲しい場合は、`.find_or_create_by!`でした。
- RubyでVO(Value Object)を作るのは、どうしたらいいんですかね。`object = Struct.new(:variables, ...)`とobjectとして使えば良い、という記述があったので、今度試してみます。←ここでいうobjectってclasなんですね。
- RubyにもJavadocのようなcomment流儀あるんですね。[YARD](http://yardoc.org)ですか。`@param [引数のクラス] 引数名 引数の説明`、`@return [戻り値のクラス] 戻り値の説明`
- 配列からhash mapを作る手法、色々ありますよね。よくある「`[key, value]`の配列の配列にしてから`.to_set`」はloopを2回回すので何だかなぁと思っていました。Rubyにもある`reduce(inject)`を使えば出来る!とあったのでやったのですけど、そうしたらrubocopに「そういう時は`each_with_object`を使え」と言われました。ナルホド、それなら確かにblockの最後に`hash`をreturnせずともよいわけですね! というわけで、`list.each_with_object({}){|value,hash|hash[key]=value}`、更に一気にhashのhashを作るには、`list.each_with_object(Hash.new{|hash,key|hash[key]={}){|value,hash|hash[key1][key2]=value}`
- 同様に、一気に`Set`を作るには、`list.each_with_object(Set.new){|value,set|set<<value}`の方が、最後に`.to_set`よりよいでしょう。
- reekに「`log.debug`を複数回使っているからまとめよ」と言われて弱りました。単純にまとめると、毎回引数を評価しちゃうじゃないですか。でもloggingなので遅延評価が必要でしょう。というわけで、blockを残せるようにmethod定義しました。`def debug(&block) Rails.logger.debug(&block); end`使う時は`debug{...}`です。
- `rescue`で`=> e`とせずとも`$!`でerrorを参照できるんですね。`rescue`で`retry`とすれば`begin`からまたやってくれるんですね! ただ、上限設けないとずっとretryし続けるので注意ですかね。
- Railsでmail送信は、configが終わっているのであとは、`ApplicationMailer`を継承したclassを作成、`app/views/class名/method名.html.haml`といったtemplateを用意、という感じでした。html mailだけだとspam判定されやすいとのことなので、`app/views/class名/method名.txt.erb`も用意しました。classのinstance fieldをmail template中で参照できます。
- ActiveRecordでの取得はなるべく`.pluck`にした方が速いし軽い、ということでそうしてますが、でもそうすると何のためのDTOなのかと。model定義の意義が少し薄れるような。modelがfatなのが悪い? `.pluck`でない場合はせめて`.select`で選択するfieldを減らしてやって生成されるobjectを少しでも軽くすると。ただActiveRecordからObjectを生成するcostが高いとのことなので、`.pluck`の方が推奨されています。...`.select`って、modelにないfieldもselectできるんですね。そしてdynamicにmethodが生えると。なんかそこまで行くと気持ち悪い気がするのは、やはりJava出身だから??
- `yield`はよく使いました。`if block_given?`も併せて。
- instanceからclass variableへの参照は`instance.class::VARIABLE`でした。
- rspecで`allow(Rails).to receive(:logger).and_return(spy('logger'))`とするとloggingが一律空回しになります。
- method chainを`allow`するには`receive_message_chain`(e.g.`expect(SomeClass).to receive_message_chain(:joins, :distinct, :where, :pluck)`) `.with(...)`を複数書いても、それぞれのmethodの引数指定ではなく、最後の一つしか評価されていない?
- rspecで、違ったparameterで同じobjectのmethodをcallするのをstubするには、`allow(...).to receive(:same_method).with(...)`を並列で何度も書けばよいです。`with(...)`をmethod chainさせるとかではないんですね。
- rspecでinstance doubleに引数を連ねるとmethod stubになります。e.g. `double('user', id: 1)`→`user.id==1`(`double`の最初の引数は任意名)
- 調べればすぐ出てくることではありますが、Railsのmigrationの`create_table`時にprimary keyを自動で出来る`id`以外にしたい場合は`create_table`の行に`primary_key: key名`と書きます。更にforeign key constraintを付け加えたい場合は、その下に`add_foreign_key :table名, :foreign_table名`と書けば大丈夫でした。
- RubyでTime format時、`.strftime('%X')`とすると一文字で`15:37:21`に、`.strftime('%F')`とすると`2020-03-01`になるので便利です。また、`.iso8601`とすると`T`を挟んでTimezone付きで`2020-03-01T15:39:10+09:00`というようにしてくれるのでこれも便利です。
- [嫁の顔忘れてもTimecop.returnは忘れないでねという話](https://daily.belltail.jp/?p=2035)にあるように、`Timecop.travel(Time.parse('2020/3/1 0:01:03')) do ... end`で囲うのはいいですね。`Timecop`も`.travel`(特定時刻にしてから時を刻む)と`.freeze`(特定時刻にして時を止める)では違うので注意です。
- Rubyで文字列の連結は`<<`が[いいみたい](https://qiita.com/Kta-M/items/c7c2fb0b61b11d3a2c48)ですね。C++みたいですね。
- ActiveRecordでN+1問題を回避するには、なるべく`joins`した方が良いようです。が、図に乗って巨大tableをjoinするとそれはcostがでかいので、分割して引いた方がいいんでしょう。`joins`するにはmodelに`has_many`とかの関係が書いてないとダメでした。2つ先の関連も書けて、`has_many :second_table, through: :first_table`と`through:`で繋げられました。
- と思ったのは幻想でした。書けちゃいますけど、joinされたものはkeyで結びついたものではなく、雑に合わさったもので、`where`で引いてもそうでないものまでついてきていて、bugを引き起こしていました。ちゃんとやらないとダメですね。
- 「ちゃんとやる」とは、[Railsでモデルを4段階joinする方法で、もう一度理解するjoinsとmerge](https://qiita.com/TeruhisaFukumoto/items/007ad22cc170d297dbcc)にあるように、ActiveRecordの`joins`で繋ぐ時に`joins(first:[second: :third])`とすること。これでちゃんとjoinされました。
- controllerで、APIっぽくheaderだけ返して中身は不要という場合、`head :created`(201)とか`head :unauthorized`(401)と書けます。
- Rubyのhere documentはもう`<<~`でよくないすか? `<<`だと左端にひっつくので。
- Rubyの`reduce`は`each_with_object`に取って代わられるのですが、`reduce(&:method)`で済むところに活路があるようです。使えたのは`Set`のmergeですね。`[Set.new([1,2,3]),Set.new([2,3,4])].reduce(&:merge)==Set.new([1,2,3,4])`
- 複数回`.include?`するなら絶対`Set`にすべきですよ。
- 定数文字列もなるべく`.freeze`すべきでしょうか。magic comment `# frozen_string_literal: true`で(が?)よいのでしょうか。
- controllerのrspecで、`before_action`のtestを書くことになったのですが、色々と悩みました。どこに書くか? 書いたclass? 使うclass? Javaでabstract classのtestだとconcrete classに書きますが、controllerは使う側に書くと多岐にわたるので、`application_controller_spec.rb`に書きました。
- curlでtest clientとしてJSONを`-d`でPOSTしたところ、どうもうまく行かなくて、調べるとserver側に`"{\"key\":\"value\"}"`なんて渡っていました。どうしてー?! HTTP Request Headerに`Contetnt-Type: application/json`が必要なんですね! そういえば。
- 今更ですがAngularJSのtestで、`window.location.href`をassertionしようとして四苦八苦しました。[How to mock $window.location.replace in AngularJS unit test?](https://stackoverflow.com/questions/20252382/how-to-mock-window-location-replace-in-angularjs-unit-test)等から得たことは、`window.location.href`だとtest(jasmin)で拾えないので`$window.location.href`にする、responseがemptyだとどうしてもtestの方でcatchしてくれないのでdummy値を`response`代入する、testでは`$window`がemptyで悩みました。`$provide.value('$window',`でprovideして`beforeEach(inject(function($injector, $rootScope, $httpBackend, $window) {`でinjectしつつ`window=$window;`と付け替えて、assertionのところで`window`を使う、という。
- `diff`で相違は出ますが、共通行を抽出するには? `comm -1 -2 file1 file2`(但しfilesはsort済であること) ref.[２つのファイルの共通行を抽出する方法](https://qiita.com/mekagazira/items/1a1791a42e435cefd5f6)
- MySQLでindexがついたかどうかを確認するには、`describe table名`では`Mul`と付くだけでよくわからず。`show create table table名`でCREATE文を出すことで。
- controllerのrspecで、sessionってどう表すのかと。`get :index,params:{aaa:'AA'}`だから`get :index,params{...},session:{...}`かと思いきや、実際そう書いてあるところもあるんですが、それではうまく行かず、`get :index,{param:'something'},{session:'something'}`と。ref. [RSpec set session object](https://stackoverflow.com/questions/22451969/rspec-set-session-object)
- rspecでprivate variableにsetするには、`some_instance.instance_variable_set('variable_name', 'some value')`、getするには`some_instance.instance_variable_get('variable_name')
- visitor patternを適用しようと思っていたのですけど、諦めました。RubyってDuck Typingだからあんまり効果を見出だせず。[Ruby で Visitor パターンをあまり使わない理由，Ruby で Visitor パターンを使うとき](https://passingloop.tumblr.com/post/13835569972/rarely-used-visitor-pattern-in-ruby)という投稿もあり。

------------------
- [upstart で start した job の設定を変更するには一度 stop する必要がある](https://abicky.net/2018/12/03/104228/)
へーそうなんだ! ってぃぅか何のための`restart`っすか? `restart`あるんだから機能すると思いますよねぇ...orz

- `1.hour.ago`があるのに`.later`や`.after`はないのでどうするんだろう? と思ったら、`1.hour.from_now`っていうんですね。
- controllerのrspec書いていて、やっと単体で通るようになったので、いざ全体的にrspecかけてみると、コケます。確かに自分の書き足したところ。でも通る時もある。どうして?→そういう、rspecの結果が不安定な時は、実行順序依存性があります。これを紐解かねばなりません。それが厄介でした。といいますか、rspecって全体的にかけると、といいますかspec fileを指定しないでdirectoryだけ指定して実行すると、spec filesの実行順序がrandomなんですね。それはその方がいいですね。なので、色んな手を使って順序依存性を解明していきます。ref. [RSpecコマンドのオプションまとめ](https://qiita.com/ShunjiKato/items/c95e6cb0fb3523aa693a) rspecに`--order random:NNNNN`を付けてrandom seedを固定、`-f d`として出力formatにclass名を出すようにし、`--no-color`でescape sequenceを抑制します。`-o filename.log`でfileに出力、それをうまく行ったときと失敗した時で保存して、class名を比較しました。当該classのtest以前だけが大事なので、そこだけ出して`sort`して`diff`取ります。失敗事例が複数取れれば、それらの共通classを`comm -1 -2 <(...) <(...)`で抽出するのが良いです。
```sh
docker-compose exec rails bash -c 'RAILS_ENV=test bundle exec rspec --order random:32510 -f d -o 32510.log --no-color'
diff <(grep -e '^[A-Za-z]' -e '^  Apis::' controllers2.log|awk '{if(/ApplicationController/){exit}print}'|sort) <(grep -e '^[A-Za-z]' -e '^  Apis::' 32510.log|awk '{if(/ApplicationController/){exit}print}'|sort)|ag '<'
comm -1 -2 <(grep -e '^[A-Za-z]' -e '^  Apis::' controllers2.log|awk '{if(/ApplicationController/){exit}print}'|sort) <(grep -e '^[A-Za-z]' -e '^  Apis::' 4627.log|awk '{if(/ApplicationController/){exit}print}'|sort)
comm -1 -2 <(grep -e '^[A-Za-z]' -e '^  Apis::' controllers2.log|awk '{if(/ApplicationController/){exit}print}'|sort) <(grep -e '^[A-Za-z]' -e '^  Apis::' 48327.log|awk '{if(/ApplicationController/){exit}print}'|sort)|grep -r -l -f /dev/stdin spec/controllers|sort -u|tr -d '\r'
```
- 単体では通るのに全体だと時々こけるというのはどうしてもよくわからないので、何度も何度も実行して観察してみると、コケる時のFの出る位置がまちまちなんですね。あぁ、これって毎回test順序違うのか、とその時初めて思い至りました。それで、rspecにおけるrandom seedの固定方法や出力形式の指定の仕方を調べ、何とか実行順を成功時と失敗時で比較して、怪しいものを試して、辿り着いたのでした。
- controllerのtestでは、`before_action`に指定したprivate methodのtest且つ利用するchild classでのtestではなくbase classでのtestなので、当初は`contorller.send('method_name')`とかやってたんですけどこれは本来的ではないな、ということで、Anonymous Controller使いました。といっても`describe`の中で`controller do ... end`して、そこで`before_action :method_name`しただけのものです。それでその`describe`内のcontrollerは定義したものになるという、お手軽な。でもここでエラー(`Error`をthrow、じゃないRubyだからraiseですか、する筈が正常に終了してしまう)が出て、悩みました。結局、わかってみればどうということはないのですが、他のspec classで、`ApplicationController.skip_before_filter :login_required`を`before do...`でやっていて、でも`after do...`で元に戻していなかったから、というのが原因でした。
- そもそも、`ApplicationController.skip_before_filter`なんてせずに、単に`allow(controller).to receive(:method)`とmockすれば十分です。そうしたら`after do...`だって不要ですし。ref. [before_filterを分離してテストする方法(Rspec)](https://qiita.com/yuutetu/items/a1586339d716aad04098) 別段`true`返す必要もないんですねこの`before_action`って。`before_filter`は古いんだそう。
- いつも`schedule`の確認をするのに。`bundle exec whenever`
- AngularJSでのredirect? になるのかな? は、routesの`$stateProvider.state('...')`に定義されたところへ、`$state.go('...')`で飛びます。ref. [Angularjs UI Routerの使い方](https://qiita.com/nogson/items/63d8f007fe987e2dfe9e) でも結局、AngularJSのrouting範囲外に飛ばしたかったので、`$window.location.href = ...`で飛ばしました。
- ActiveRecordの`scope`に引数を取る場合には、`scope scope_name ->(arg1, arg2) { ... }` 矢印とカッコはくっつけないとreekに怒られます。
- Rubyで`String`の`select`を正規表現でするなら`grep(/.../)`、`-v`の場合は`grep_v`([Ruby equivalent to grep -v](https://stackoverflow.com/questions/4165971/ruby-equivalent-to-grep-v/33881808))
- AngularJSのroutingってURLについたanchor(`#`以下)に出るのですが、それってそもそもbrowserがserver側に送信しないんですね...
- ActiveRecordで選択して保存を1行で。例えば`User.find(1).update(name:'new name')`
- rspecでActiveRecordのtest dataってfixtureやFactoryGirlを使うものかと思っていたのですけど、普通に`.create`とか`.update`とかして大丈夫なんですね。大丈夫、というのは、後腐れがない、んですね。
- RSpecで`allow`でmockした時に、`.and_return`だけでなく処理(`sleep(1)`)を入れたかったんですけどどうしたら? と思ったら、単にblockでいいんですね。blockの最終評価値がreturnになる、というなら`.and_return`って不要なの?

<!--
```sh
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/adset?campaign_id=6119375093538&date_preset=customize&advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/campaign?campaign_id=6119375093538&advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/adset?adset_id=6119375094338&advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/adset?adset_id=6119375094338&type=daily&advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/adgroup?adset_id=6119375094338&advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/stats/adgroup?type=daily&advertiser_id=3&adgroup_id=6119375096938'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/adgroups/show/6119375096938?advertiser_id=3'
curl -v -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/adgroups/reach_estimateand_description?adgroupid=6119375096938&advertiser_id=3'
curl -v -H 'content-type:application/json' -d '{"creative_id":"6119375095738"}' -H "JWT-Authorization:$JWT" -H "X-CSRF-Token:$CSRF" -H "Cookie:$COOKIE" 'http://dev.ibiza.jp:13000/apis/ads/preview?advertiser_id=3'
```
-->
