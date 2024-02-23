---
title: MinAvgTwoSlice
tags:
  - PHP
  - '#Codility'
private: false
updated_at: '2019-01-22T21:37:49+09:00'
id: 8ec70caf5ac7b9b8e8e4
organization_url_name: null
slide: false
ignorePublish: false
---
内容理解は出来たけど、解くのが難しそう、というのが第一印象。二重ループが許されるなら正確性に
ついては満足させられると思うけど、計算量は `O(N)` 指定なので、不可。よって何かしら工夫が必要。
でも、とりあえず問題の理解が合っとるか確認するために、二重ループにて回答を作成した。

## [1st][a]

<table>
    <tr><th>Correctness</th><th>Performance</th><th>Task Score</th></tr>
    <tr>
        <td>100%</td>
        <td>20%</td>
        <td>60%</td>
    </tr>
</table>

```php
function solution($A) {
    $start_position = 0;
    $min_avg = (($A[0] + $A[1]) / 2);
    
    for($i = 0, $sum = 0, $avg = 0; $i < count($A)-1; $i++) {
        $sum = $A[$i];
        for($j = $i+1; $j < count($A); $j++) {
            $sum += $A[$j];
            $avg = $sum / ($j - $i + 1);
            if($min_avg > $avg) { 
                $min_avg = $avg;
                $start_position = $i;
                }
        }
    }
    
    return $start_position;
    
}
```

ノーテストで上手くいったから、気持ち良かった。思わずこれで終わろうとした。でも、本題はここから。
いかに `O(N)` で処理するか。

・・・なのだが、一日で解決しきれなかったので、解答を見る。

## 解答
[いつもの jimii さん][b]のものを解答としました。Thx.

<table>
    <tr><th>Correctness</th><th>Performance</th><th>Task Score</th></tr>
    <tr>
        <td>100%</td>
        <td>100%</td>
        <td>100%</td>
    </tr>
</table>

```php
function solution($A) {
   $length = count($A);
   $min_average = ($A[0] + $A[1]) / 2.0;
   $position = 0;
   if ($length==2) return 0;
   for ($i=2 ; $i < $length; $i++) {
       $temp1 = ($A[$i-1] + $A[$i]) / 2.0;
       $temp2 = ($A[$i-2] + $A[$i-1] + $A[$i]) / 3.0;
       if ($temp1 < $min_average) {
           $position = $i - 1;
           $min_average = $temp1;
       }
       if ($temp2 < $min_average) {
           $position = $i - 2;
           $min_average = $temp2;
       }
    }
    return $position;
}
```

今回、新たな発見を２つした。
一つは、[このページ][c]に書いてある。

> Codilityの例題PDFにかかれたprefix sumのアルゴリズムの工夫だけでとけるのかな？と思って、
> ずっと考えてたのですがどうやら違いそうだという事でググってしまいました。

**例題 PDF**？確かに、Lesson ごとに PDF があるのは知っとったけど、それに何か意味が？
と疑問に思いつつ見てみると、**Lesson に取り組む前に必読のものだ**ということが分かった。
（ただ、Prefix/Suffix sum 自体の概念が難しいため、要勉強・・・。）

もうひとつは、単に、この問題を解く際、**４つ以上のスライスを考慮する必要がないこと**。
つまり、２つの場合と３つの場合で最小となる平均値が全体でみても最小平均値。
ただ、飲み込めないのは、**「２つ／３つに分解した際の最小平均値より、分解する前の平均値の方が
小さいことはないのか？」**という疑問。

これは、問題の前提を考えれば分かる。
例えば、[1, 2, 3, 4] という配列の場合、`(1 + 2 + 3 + 4) / 4 = 2.5` であり、分解したスライスで
見ると、左部分は `(1 + 2) / 2 = 1.5` 、右部分は `(3 + 4) / 2 = 3.5` となる。
よって、この場合は左部分が最小の平均値を持つスライスであり、問題の答えは 1 になるのだが、
0 ≦ a ≦ b ≦ n (a は始点、b は終点、n は要素数) の条件で考えれば、上記の数列は考えられる
最小の数列であり、・・・。この後の説明が思いつかないし、これで合っているのかも怪しいが、
要は**４つ以上のスライスは２つまたは３つのスライスに分解でき、分解後のそれぞれのスライスの
平均値は、分解前のそれより必ず小さい**そうだ。なので、jimii さんの解き方のように、
`$A[$i-2]` から `$A[$i]` の範囲で２つまたは３つのスライスを `$i` を増分しながら計算し、
最小の平均値となったスライスの始点を返り値とすれば良い、のである。
恐らく `prefix sum` の大事な概念が使われていると思うが、今は理解出来ていないのでパスである。


悔しくて jimii さんのコードの `for` 文を `foreach` に置き換え、`$second_last` と `$last` なる
変数を用意し、ちょっと意味を分かりやすくしようとしたものの、明らかに jimii さんのように、
`$A[$i-2]`, `$A[$i-1]` と表記した方が分かりやすいしスッキリするしで、改変は諦めた。無念。

<!-- 以下、参考 URL -->
[a]: https://app.codility.com/demo/results/trainingUDJ8ER-VMG/
[b]: https://github.com/jimii/codility/blob/master/MinAvgTwoSlice/php/MinAvgTwoSlice.php
[c]: http://codility-lessons-jp.blogspot.jp/2014/07/lesson-3-minavgtwoslice.html
