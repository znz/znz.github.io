---
layout: post
title: "OSS Gate Workshopのブランチ名の調査"
date: 2017-08-15 20:00:00 +0900
comments: true
categories: oss-gate
---
[OSS Gate ワークショップのリポジトリー](https://github.com/oss-gate/workshop)では、ワークショップの最後にアンケートを pull request で集めているのですが、
tutorial/retrospectives/README.md には https://github.com/oss-gate/workshop/commit/0d12651d72f8bcec0cd550da6a4e97d25caa3968 で「適当なブランチ名」と追加され (今は [#549](https://github.com/oss-gate/workshop/pull/549) で `add-retrospective` に変更済み)、
以前は全く指針などはなかったので、
みんなどんなブランチ名を使っていたのか気になったので、
集計してみました。

<!--more-->

## 集計スクリプト

簡単のため、`git log` の出力から日付などの可変部分はできるだけまとめて集計するようにしてみました。

```
#!ruby
count = Hash.new(0)
ARGF.each_line do |line|
  if /Merge pull request #\d+ from / =~ line
    id, branch = $'.split('/', 2)
    branch.sub!(Regexp.quote(id), "{id}")
    branch.sub!(/\d{4}(.?)\d{2}(.?)\d{2}/, 'YYYY\1MM\2DD')
    branch.sub!(/tokyo|osaka|pixiv/, '{place}')
    count[branch] += 1
  end
end
count.keys.sort_by do |k|
  [-count[k], k]
end.each do |k|
  puts "#{count[k]}:#{k}"
end
```

## 集計結果

ふりかえり以外の pull request もあるので、すべてがふりかえりのものとは断言できないのですが、
圧倒的に master, patch-1, patch-2 が多かったです。

master は別ブランチを作っていないことが多いということを示しています。

patch-1, patch-2, patch-3 などは github の Web から Edit で編集を開始するとできるブランチ名だったはずなので、
Web で操作して作成した pull request かもしれません。

`masayuki14_closed_workshop` は「社内向けのクローズドでWorkshopを開催したので、ふりかえりのアンケートを共有します」とあるので、
特殊なので無視して良さそうです。

形式がバラバラで集計ではバラバラのままにしていますが、日付・場所・ID・役割 (beginner や mentor など) が入っていることが多く、
workshop (や meetup)、retrospective (answer, comment, enquete) や add などの単語が使われていることが多いようです。

```
% git log | ruby /tmp/s.rb
99:master
60:patch-1
14:patch-2
6:masayuki14_closed_workshop
5:{id}
4:YYYY-MM-DD-{place}
4:YYYYMMDD_{place}_meetup_{id}
4:patch-3
3:retrospectives-YYYY-MM-DD
3:{id}-patch-1
2:YYYY-MM-DD-{id}
2:YYYYMMDD-{place}-workshop
2:YYYYMMDD_{place}_{id}
2:add-{id}-comment
2:knokmki612-patch-1
1:Add-{id}-enquete
1:YYYY-MM-DD-{place}-{id}
1:YYYY-MM-DD-{place}-{id}-answer
1:YYYY-MM-DD-{place}_{id}_retrospective
1:YYYYMMDD
1:YYYYMMDD_oss_gate_{place}_{id}_enquete
1:YYYYMMDD_retro
1:YYYYMMDD_ryo-endo
1:YYYYMMDD_{id}
1:YYYYMMDD_{place}_mentor_{id}
1:YYYYMMDD_{place}_{id}_answer
1:add-YYYYMMDD-{id}
1:add-YYYYMMDD-{place}
1:add-cm-wada-yusuke
1:add-comment
1:add-mentor-tSU-RooT-11-26
1:add-questionnaire-desc
1:add-yaml
1:add-{id}-0116
1:add-{id}-YYYY-MM-DD
1:add_beginner-{id}
1:add_description_of_oss_gate
1:add_mentor_yaml_by_{id}
1:add_questionnaire
1:add_retrospective_{id}
1:aggeregate
1:answer_by_satoryu
1:begginer-n-ina
1:beginner-{id}
1:change_issue_template
1:enquete
1:feature/YYYY_MM_DD_{place}_supporter_{id}
1:feature/add_{id}_answer
1:feature/retrospective
1:fill_retrospective_{id}
1:fix/issue_26
1:fix/retrospective_27
1:fix/typo
1:fix_retrospective
1:fix_supporter_questionnaire
1:fix_survey_url
1:fix_yaml_error
1:gitter-badge
1:goto_add_comment
1:hiroysato
1:improve_slide_2
1:improve_slide_split_2
1:issue_template
1:issue_template-2
1:kamimura
1:link-to-tutorial
1:make_template
1:mentor-majima
1:mentor-matsue
1:my_retrospectives
1:opinionaire_YYYYMMDD_morioka
1:patch-5
1:question_template
1:questionnaire
1:questionnaire-YYYY-MM-DD-{place}
1:rename
1:result_of_YYYY_MM_DD
1:retrospective
1:retrospective-{id}
1:retrospective_instruction
1:retrospective_{id}
1:retrospectives/{id}
1:sort_retrospectives
1:stat-rate
1:support-yaml-with-bom
1:survey_2017_3_25
1:tdtds
1:tricknotes
1:tricknotes-patch-1
1:update-retrospectives-readme
1:update_for_2016_9_24
1:write_questionnaire_kohei_otsuka
1:write_retrospectives
1:y-goto_surver_YYYYMMDD
1:y-goto_survey_YYYYMMDD
1:{id}-enquete-170116
1:{id}-mentor
1:{id}-oss-comment
1:{id}-questionnaire
1:{id}-workshop-answer1
1:{id}/n-n-n-
1:{id}/retrospective
1:{id}1
1:{id}_ans
1:{id}_closed_workshop
1:{place}_0629
```

## まとめ

今回は [#549](https://github.com/oss-gate/workshop/pull/549) の時に目視で確認していたのを、もうちょっとちゃんと確認してみました。
ブランチ名やコミットメッセージはみんな色々悩んでつけている感じがして、過去のものを眺めてみると面白いと思います。

OSS Gate のワークショップのふりかえりはワークショップの特性上、できるだけブランチ名も悩まずにとりあえずこれを使ってもらう、というのがあるのが望ましいので、固定の説明に変更しましたが、慣れていれば独自のブランチ名やコミットメッセージでも全く問題なさそうです。
