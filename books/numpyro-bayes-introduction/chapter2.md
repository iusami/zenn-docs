---
title: "chapter2:ベイズ統計学の基本原理"
---

※ 書籍の第1章は前書きなのでスキップ

## 未知の比率に対する推論
内閣支持率はどの程度と推測できるか？という問題を考える。全ての投票者のうち確率$q$で内閣支持者が存在するとすれば、

$$
{\rm Pr}\{X_i=x_i\} =  \left \{
\begin{array}{l}
q　(x＝1) \\
1-q　(x=0)
\end{array}
\right.
$$

とかける。ここで$x=1$がある回答者が支持者であることを表す。  
まとめると、  

$$
{\rm Pr}\{X_i=x_i\} = q^{x_i}(1-q)^{1-x_i}=p(x_i|q)
$$

とかける。この時、$p(x_i|q)$は確率関数と呼ばれる。この確率関数で定義される分布はベルヌーイ分布と呼ばれる。

ベイズ統計学においては、上記の未知のパラメータである$q$を確率変数であると解釈して、パラメータの統計的推論を行う。  
ここで出発点となる確率分布として事前分布というものを想定し、データが持つパラメータに関する情報で更新して事後分布を求めてパラメータの値に関数する推論を行う。  
ここでの事前分布はパラメータに関する確率分布であって、データが生成された分布とは異なるものであることに注意する。
どのような事前分布を用いるかは分析者が持つ事前情報によって決定される。例えば、以下の確率関数で表される一様分布を用いれば、分析者が特段$q$に対する事前情報を持っていないため、「$q$がとりうる範囲$0\leqq q \leqq 1$の全ての値に対して真である可能性は等しい」ということになる。

$$
p(x) =  \left \{
\begin{array}{l}
\frac{1}{b-a}　(a\leqq x \leqq b) \\
0　(x< a, b<x)
\end{array}
\right.
$$

ただしここで考えている問題を考慮すれば、支持率が90%を越えることは考えづらいので、その情報を表せる事前分布を考える。

ここで以下の確率密度関数で与えられるベータ分布を考える。

$$
p(x|\alpha,\beta) = \frac{x^{\alpha-1}(1-x)^{\beta-1}}{B(\alpha,\beta)}  
$$
ただし、$B(\alpha,\beta)$は

$$
B(\alpha, \beta)=\int_{0}^{1} x^{\alpha-1}(1-x)^{\beta-1} dx 
$$

一様分布、ベータ分布の形状を確認するため、numpyroでサンプリングを行ってみる。



```python
from jax.random import PRNGKey
from numpyro.distributions import Uniform, Beta
#keyの準備
key = PRNGKey(seed=1234)

#一様分布とベータ分布からのサンプリング
uni = Uniform()
beta = Beta(concentration0=4, concentration1=6)
uniform_sample = uni.sample(key, sample_shape=(10000,))
beta_sample = beta.sample(key, sample_shape=(10000,))
uniform_df = pd.DataFrame({"value":uniform_sample, "kind":"uniform"})
beta_df = pd.DataFrame({"value":(beta_sample), "kind":"beta"})
sample_df = pd.concat([uniform_df, beta_df], axis=0).reset_index(drop=True)

#seabornで可視化
sns.histplot(data=sample_df,x="value", hue="kind", bins=100,binrange=(0,1))
```
![](https://storage.googleapis.com/zenn-user-upload/2184ef266004-20211228.png)

一様分布はまさに一様に広がっていて、ベータ分布は山形の分布をしていることがわかる。

## ベイズの定理による事後分布の導出
事後分布の導出にあたって、まずはベイズの定理を導出する。  
データを$D=(x_1,...x_n)$と表記した時、成功確率$q$と$D$の組み合わせが同時に実現する確率分布を$p(q,D)$と表記する。  
条件付き確率分布の性質

$$
p(q|D) = \frac{{p(q,D)}}{p(D)}
$$
から

$$
\begin{equation}
p(q,D) = p(q|D)p(D) = p(D|q)p(q)    
\end{equation}
$$
より、

$$
\begin{equation}
p(q|D)=\frac{p(D|q)p(q)}{p(D)}
\end{equation}
$$

もう少し$p(D|q)$について考えると、$q$が与えられた時の$D$の確率分布とは、$D$を生成した確率分布そのものになる。かつ、前提としている問題の$D$の要素はベルヌーイ分布に従って独立に生成されているため、以下の通りにかける。

$$
\begin{split}
p(D|q)&={\rm Pr}\{X_1=xi, X_2=x_2,...X_n=x_n\} \\
&=\prod_{i=1}^{n}{\rm Pr}\{X_i=x_i|q\} \\
&=\prod_{i=1}^{n}q^{x_i}(1-q)^{1-x_i} \\
&=q^{x_1}(1-q)^{1-x_1}q^{x_2}(1-q)^{1-x_2}...q^{x_n}(1-q)^{1-x_n} \\
&=q^{(x_1+...+x_n)}(1-q)^{(n-x_1-...-x_n)}\\
&=q^y(1-q)^{n-y},  y=\sum_{i=1}^{n}x_i
\end{split}
$$

一方、$p(D)$に関しては、同時分布を周辺化したものと考えられるので、

$$
p(D)=\int_{0}^{1}p(D,q)dq=\int_{0}^{1}p(D|q)p(q)dq
$$

となり、$p(D|q)$を$p(q)$で平均したものとわかる。  
ベイズ統計では、$p(D|q)$を尤度、$p(D)$を周辺尤度という。  
再び(2)式に戻り、$p(D)$がデータ観測後には固定されてしまうことから、

$$
p(q|D)\varpropto{p(D|q)p(q)}
$$

と書くことができる。つまり、事後分布は尤度と事前分布の席に比例すると言える。  
ここで事前分布にベータ分布（$\textit{Be}(\alpha, \beta)$)を用いる。また、ハイパーパラメータを$\alpha=\alpha_0, \beta=\beta_0$とし、$q$の事前分布が$\textit{Be}(\alpha_0, \beta_0)$であることを

$$
q\sim \textit{Be}(\alpha_0, \beta_0)
$$

と書く。  
ここでベイズの定理を適用すると、

$$
\begin{split}
p(q|D)&\varpropto{p(D|q)p(q)}\\
&\varpropto q^y(1-q)^{(n-y)}\frac{q^{\alpha_0-1}(1-q)^{\beta_0-1}}{B(\alpha_0,\beta_0)}\\
&\varpropto q^{(y+\alpha_0-1)}(1-q)^{(n-y+\beta_0-1)}
\end{split}
$$

ここで比例記号を消すために、基準化定数$\kappa$を用意して、

$$
p(q|D)=\kappa \times q^{(y+\alpha_0-1)}(1-q)^{(n-y+\beta_0-1)}
$$

と書く。$p(q|D)$が確率密度関数になるには、$\int_{0}^{1}p(q|D)dq=1$となる必要があるため、

$$
\begin{split}
\kappa &= \frac{1}{\int_{0}^{1}q^{(y+\alpha_0-1)}(1-q)^{(n-y+\beta_0-1)}dq}\\
&= \frac{1}{B(y+\alpha_0, n-y+\beta_0)}
\end{split}
$$

となり、まとめると

$$
\begin{split}
p(q|D)&=\frac{q^{(y+\alpha_0-1)}(1-q)^{(n-y+\beta_0-1)}}{B(y+\alpha_0, n-y+\beta_0)}\\
&=\frac{q^{(\alpha_\ast-1)}(1-q)^{(\beta_\ast-1)}}{B(\alpha_\ast, \beta_\ast)}, \alpha_\ast=y+\alpha_0,\beta_\ast=n-y+\beta_0
\end{split}
$$

となり、ベータ分布になっていることがわかる。  
従って、成功確率がベルヌーイ分布に従っている時、ベータ分布を事前分布にすると事後分布もベータ分布になる。このような事前分布は自然共役事前分布という。

これで事後分布を計算する準備が整ったので、numpyroで実装してみる。

```python
from numpyro.distributions import BernoulliProbs
bern = BernoulliProbs(0.25)
#データの準備
n = [10,50,250]
bern_sample = [bern.sample(key, sample_shape=(i,)) for i in n]

#事後分布の計算関数の準備
def beta_posterior_parameter(alpha, beta, sample):
  y = np.sum(sample)
  alpha_ast = y + alpha
  beta_ast = len(sample) - y + beta
  return alpha_ast, beta_ast

alpha0=1
beta0=1

post_params = [beta_posterior_parameter(alpha0, beta0, np.array(i)) for i in bern_sample]

#生成したサンプルごとの事後分布の計算
beta_posterior_samples = []
for ct, params in enumerate(post_params):
  alpha_curr, beta_curr = params
  beta_post = Beta(alpha_curr, beta_curr)
  sample_df_tmp = pd.DataFrame({"value":np.array(beta_post.sample(key, sample_shape=(10000,))), "kind":"params{}".format(str(ct))})
  beta_posterior_samples.append(sample_df_tmp)
beta_posterior_samples_df = pd.concat(beta_posterior_samples).reset_index(drop=True)

#事前分布のサンプル
beta_pre = Beta(alpha0, beta0)
beta_pre_sample = np.array(beta_pre.sample(key, sample_shape=(10000,)))
beta_pre_sample_df = pd.DataFrame({"value":beta_pre_sample,"kind":"prior"})

concat_df = pd.concat([beta_posterior_samples_df, beta_pre_sample_df]).reset_index(drop=True)

#seabornで可視化
sns.histplot(concat_df, x="value", hue="kind", bins=100,binrange=(0,1))
```

![](https://storage.googleapis.com/zenn-user-upload/f02388fcdfe5-20211228.png)

この例では事前分布を一様分布($\alpha=1,\beta=1$のベータ分布)とした時の事後分布を計算している。params0,1,2はそれぞれq=0.25のベルヌーイ分布から10,50,250のサンプルを生成した場合に求められた事後分布のベータ分布のパラメータを意味する。  
図からサンプルサイズを大きくするほど、事後分布が尖っていくように見え,
q=0.25の可能性が高いということを示している。事前分布が一様分布であったとしても、サンプルサイズがある程度大きければ事前分布の影響をある程度無視できることがわかる。  
続いて、異なるハイパーパラメータを用いたベータ分布($\alpha=6,\beta=4$のベータ分布)を事前分布とした時に事後分布を以下に表示する。

![](https://storage.googleapis.com/zenn-user-upload/f568b5b21290-20211228.png)

事前分布が右に偏った分布のため、先ほどよりは右に引っ張られたような事後分布がどのサンプルサイズにおいても生成されている。

## 未知のパラメータに関する推論
事後分布を求めることができたので、その図を見れば$q$の真の値に関して推論することはできるが、実際にはグラフではなく数値として示す必要があることもある。事後分布からこの具体的な値を求めることを点推定という。ベイズ統計学における点推定は、「パラメータの真の値の候補の中から真の値として最も相応しい値を一つ選択すること」になる。そこで、点推定の値とパラメータの真の値の乖離を規準（損失関数）で測ることで決定する。以下ではパラメータの真値を$q$、$q$の点推定の値を$\delta$、損失関数を$L(q,\delta)$と表記する。代表的な損失関数は以下の通り。

$$
x = \begin{cases}
   (q-\delta)^2 &(2乗損失) \\
   |q-\delta| &(絶対損失) \\
   1-\bold{1}_q(\delta) &(0-1損失)
\end{cases}
$$
ここで$\bold{1}_q(\delta)$は

$$
\bold{1}_q(\delta) = \begin{cases}
1, &(\delta=q) \\
0, &(\delta \not ={q})
\end{cases} 
$$

という指示関数である。それぞれの損失関数に対応する点推定は、

$$
x = \begin{cases}
   事後分布の平均 &(2乗損失) \\
   事後分布の平均 &(絶対損失) \\
   事後分布の最頻値 &(0-1損失)
\end{cases}
$$
になる。

点推定はそれぞれの損失関数を最小にするという意味でパラメータの真の候補に相応しいが、パラメータの真の値に関して不確実性が残るので、1つの値のみを推定値としてしまうのは不安が残る。そこで、真の値が入っている可能性が高い区間を提示するという方法もとられる。これを区間推定という。区間推定の一例として、HPD（highest posterior density）区間を紹介する。  
一般に100(1-c)%HPD区間は、区間内の点の確率密度が区間外の確率密度よりも必ず高くなる区間として定義される。  
特に単峰の分布ならば、

$$
\begin{split}
{\rm Pr}\{a_c\leqq q \leqq b_c|D \} &= 1-c \\
p(a_c|D)&=p(b_c|D)
\end{split}
$$
なる区間$[a_c,b_c]$と定義される。

ここでnumpyroを使って点推定と区間推定の例を挙げる。numpyroでの推定結果の表示は簡単で、`numpyro.diagnostics.summary`関数にサンプルを引数として与えるのみで良い。以下にコードを示す。

```python
from numpyro.diagnostics import summary
beta_hpd = Beta(6,4)
beta_hpd_sample = beta_hpd.sample(key, sample_shape=(10000,))
summary_res = summary(beta_hpd_sample,group_by_chain=False)
summary_df = pd.DataFrame(summary_res)
summary_df.columns=["値"]
```

これを実行すると、以下のようなデータフレームを得られる。

|    項目   |   値         |
| ----  | ----       |
|5.0%	|[0.36574516]|
|95.0%	|[0.8501521]|
|mean	|[0.59987605]|
|median	|[0.60853326]|
|n_eff	|[9889.899877001562]|
|r_hat	|[0.9999139]|
|std	|[0.14782001]|

5%,95%はHPD区間の左端、右端の値を意味する。summaryの引数のprobの値がデフォルトでは0.9となっていて、90%HPDを返すためである。その他平均値、中央値、最頻値、分散等を返却する。n_effは実効標本数、r_hatはGelman-Rubinの収束判定（説明は第4章に譲る）を意味する。

実際にサンプルとHPD区間の両端を図示する。

```python
sns.histplot(beta_hpd_sample.reshape(-1))
plt.vlines(summary_res["Param:0"]["5.0%"],0,10000,color="red")
plt.vlines(summary_res["Param:0"]["95.0%"],0,10000,color="red")
plt.ylim(0,1500)
```

![](https://storage.googleapis.com/zenn-user-upload/27d89441d8f7-20211229.png)

## 仮説検証の方法
ここからは仮説検証の説明を行う。統計分析におけるパラメータに関する仮説は、パラメータが取りうる範囲として定義される。例えば、
- ある範囲内の値をとる （{$q:0.5 \leqq q \leqq 1$}）
- 特定の値に等しい （$q=0.5$）
- 特定の値に等しくない ($q \not ={0.5}$)

などである。パラメータ$q$のとりうる範囲$S_i(i=0,1,2,...)$で定義される仮説$H_i$は

$$
H_i:q\in S_i
$$
と表記される。
ベイズ統計学では、$q$の事後分布$p(q|D)$において、$H_i$が成り立つ事後確率

$$
{\rm Pr}\{H_i|D\}={\rm Pr}\{q\in S_i|D\}=\int_{S_i}p(q|D)dq
$$

を評価することで、仮説$H_i$の妥当性を評価する。上記の事後確率がほとんど1なら仮説が成立すると言っていいし、ほぼ0なら成り立っていないと言っていい。
また、事後オッズ比で比較することもある。例えば、以下のような2つの仮説の候補があるとする。

$$
x = \begin{cases}
   H_0 &q\in S_0 \\
   H_1 &q\in S_1
\end{cases}
$$

通常は$S_0\cap S_i = \empty$かつ$S_0\cup S_1 = [0,1]$とする。つまり必ずどちらかの仮説が正しいとする。この時事後オッズ比は

$$
事後オッズ比=\frac{{\rm Pr}\{H_0|D\}}{{\rm Pr}\{H_1|D\}} = \frac{{\rm Pr}\{q\in S_0|D\}}{{\rm Pr}\{q\in S_1|D\}}
$$
と定義される。ここで内閣支持率が30％を下回っているかどうかは、

$$
x = \begin{cases}
   H_0 &q \geqq 0.3 \\
   H_1 &q < 0.3
\end{cases}
$$

という2つの仮説を比較すればわかる。従って、${\rm Pr}\{q\geqq 0.3|D\}/{\rm Pr}\{q< 0.3|D\}$を計算すればよい。
ただし、事後分布は事前分布に引きずられることがわかっているので、事前分布がどちらかの仮説に有利な設定であった場合、事後オッズ比もその仮説に有利なものが得られてしまうかもしれない。その場合はベイズ・ファクターを使用する。仮説$H_0$と$H_1$を比較するベイズ・ファクター${\rm B}_{01}$は

$$
{\rm B_{01}} = \frac{{\rm Pr}\{H_0|D\}}{{\rm Pr}\{H_1|D\}} \div \frac{{\rm Pr}\{H_0\}}{{\rm Pr}\{H_1\}}
$$

として定義される。ここで

$$
事前オッズ比=\frac{{\rm Pr}\{H_0\}}{{\rm Pr}\{H_1\}}=\frac{{\rm Pr}\{q\in S_0\}}{{\rm Pr}\{q\in S_1\}}=\frac{\int_{S_0}p(q)dp}{\int_{S_1}p(q)dp}
$$

であることから、ベイズ・ファクターは事後オッズ比と事前オッズ比の比率になっている。この式から、ベイズ・ファクターは仮説$H_1$がデータ$D$によってどれだけ補強されたかを測っていると言える。

次に、パラメータの真の値が特定の値に等しいかどうかの仮説検証を考えてみる。つまり、

$$
\begin{cases}
   H_0 &q = q_0 \\
   H_1 &q \not ={q_0}
\end{cases}
$$

という仮説の比較である。しかし、$q$の事前分布に連続な確率分布を用いると${\rm Pr}\{q=q_0\}$=${\rm Pr}\{q=q_0|D\}=0$となって事前オッズ比も事後オッズ比も0になってしまう。そこで、

$$
p(q)=p_0\delta(q-q_0)+(1-p_0)f(q),\ \ 0<p_0<1
$$
という事前分布を考える。$\delta(\cdot)$はディラックのデルタ関数。
- 任意の連続関数$g(x)$に対して$\int_{-\infin}^{\infin}g(x)dx=g(0)$
- $\int_{-\infin}^{\infin}\delta(x)dx=1$
- $x \not ={0}のとき\delta(x)=0$ 
という性質を持つ。
この事前分布を使ったとき、事前オッズ比は

$$
\frac{{\rm Pr}\{q=q_0\}}{{\rm Pr}\{q\not ={q_0}\}}=\frac{p_0}{1-p_0}
$$

$q$の事後分布は

$$
\begin{split}
p(q|D) &= \frac{p(D|q)p(q)}{\int_{0}^{1}p(D|q)p(q)dq} \\
&=\frac{p(D|q)\{p_0\delta(q-q_0)+(1-p_0)f(q)\}}{\int_{0}^{1}p(D|q)\{p_0 \delta (q-q_0)+(1-p_0)f(q) \}dq}\\
&=\frac{p_0p(D|q)\delta(q-q_0)+(1-p_0)p(D|q)f(q)}{p_0\int_{0}^{1}p(D|q)\delta(q-q_0)dq+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}\\
&=\frac{p_0p(D|q)\delta(q-q_0)+(1-p_0)p(D|q)f(q)}{p_0p(D|q_0)+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}
\end{split}
$$
となり、

$$
\begin{split}
{\rm Pr}\{q=q_0|D\}&=\frac{p_0p(D|q)}{p_0p(D|q_0)+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq} \\
{\rm Pr}\{q\not ={q_0}|D\} &= 1- {\rm Pr}\{q=q_0|D\} \\
&=\frac{p_0p(D|q_0)+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq-p_0p(D|q_0)}{p_0p(D|q_0)+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}\\
&=\frac{(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}{p_0p(D|q_0)+(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}
\end{split}
$$

となる。よって事後オッズ比は

$$
\frac{{\rm Pr}\{q=q_0|D\}}{{\rm Pr}\{q\not ={q_0}|D\}}=\frac{p_0p(D|q_0)}{(1-p_0)\int_{0}^{1}p(D|q)f(q)dq}
$$

従って、ベイズ・ファクターは

$$
\begin{split}
    
{\rm B_{01}} &= \frac{{\rm Pr}\{q=q_0|D\}}{{\rm Pr}\{q\not ={q_0}|D\}} \div \frac{{\rm Pr}\{q=q_0\}}{{\rm Pr}\{q\not ={q_0}\}}\\
&=\frac{p_0p(D|q_0)}{(1-p_0)\int_{0}^{1}p(D|q)f(q)dq} 
\times\frac{1-p_0}{p_0}\\
&=\frac{p(D|q_0)}{\int_{0}^{1}p(D|q)f(q)dq}
\end{split}
$$

分子分母に$f(q_0)$をかけると

$$
\begin{split}
{\rm B_{01}} &= \frac{p(D|q_0)}{\int_{0}^{1}p(D|q)f(q)dq}
 \times \frac{f(q_0)}{f(q_0)}\\
 &=\frac{p(D|q_0)f(q_0)}{\int_{0}^{1}p(D|q)f(q)dq} \times \frac{1}{f(q_0)}\\
 &=\frac{p(q_0D)}{f(q_0)}

\end{split}
$$

このような形のベイズ・ファクターはSDDR(Savage-Dickey Density Ratio)と呼ばれる。

## 将来の確率変数の値の予測
最後に、将来の確率変数の予測の方法を示す。  
ベルヌーイ分布から将来に観測されるであろう値を$\tilde{x}$とすると、

$$
p(\tilde{x},x_1,...,x_n)=p(\tilde{x},D)
$$

というデータ$D$との同時確率分布と考えられる。  
条件付き確率分布の性質から、

$$
p(\tilde{x},D)=p(\tilde{x}|D)p(D) \rArr p(\tilde{x}|D)=\frac{p(\tilde{x},D)}{p(D)}
$$

ここで、$p(D)$は周辺尤度とみなせるので、

$$
p(D)=\int_{0}^{1}p(D|q)p(q)dq
$$

さらに、$p(\tilde{x}, D)$の$\tilde{x}$が観測された後の尤度とみなせるので、

$$
p(\tilde{x}, D)=\int_{0}^{1}p(\tilde{x},D|q)p(q)dq
$$

まとめると、

$$
p(\tilde{x}|D)=\frac{\int_{0}^{1}p(\tilde{x},D|q)p(q)dq}{\int_{0}^{1}p(D|q)p(q)dq}
$$

これがデータ$D$を与えられた下での$\tilde{x}$の条件付き確率分布であり、$\tilde{x}$の予測分布という。特に$\tilde{x}$と$D$が互いに独立ならば、

$$
p(\tilde{x},D|q)=p(\tilde{x}|q)p(D|q)
$$

なので、

$$
\begin{split}
p(\tilde{x}|D)&=\frac{\int_{0}^{1}p(\tilde{x}|q)p(D|q)p(q)dq}{\int_{0}^{1}p(D|q)p(q)dq}\\
&=\int_{0}^{1}p(\tilde{x}|q)\frac{p(D|q)p(q)}{\int_{0}^{1}p(D|q)p(q)dq}dq\\
&=\int_{0}^{1}p(\tilde{x}|q)\frac{p(D|q)p(q)}{p(D)}dq \\
&=\int_{0}^{1}p(\tilde{x}|q)p(q|D)dq
\end{split}
$$

この式はベルヌーイ分布の期待値を事後分布で評価したものという意味になる。ざっくり言えば、分布やモデルがわからない時は、事後分布で平均すると考えれば良い。

今考えている問題では確率変数はベルヌーイ分布から独立に生成されているとしているので、ベルヌーイ分布の予測分布を上記の式に則って求められる。

$$
\begin{split}
p(\tilde{x}|D)&=\int_{0}^{1}q^{\tilde{x}}(1-q)^{1-\tilde{x}}\frac{q^{\alpha_\ast-1}(1-q)^{\beta_\ast-1}}{B(\alpha_\ast,\beta_\ast)}dq\\
&=\frac{\int_{0}^{1}q^{\tilde{x}+\alpha-1}(1-q)^{\beta_\ast-1}}{B(\alpha_\ast,\beta_\ast)}\\
&=\frac{B(\alpha_\ast+\tilde{x},\beta_\ast-\tilde{x}+1)}{B(\alpha_\ast,\beta_\ast)}
\end{split}
$$

ここで、ベータ関数の性質

$$
B(\alpha+1,\beta)=\frac{\alpha}{\alpha+\beta}B(\alpha,\beta),  B(\alpha,\beta+1)=\frac{\beta}{\alpha+\beta}B(\alpha,\beta)
$$

と用いると、

$$
p(\tilde{x}=1|D)=\frac{B(\alpha_\ast+1,\beta_\ast)}{B(\alpha_\ast,\beta_\ast)}=\frac{\alpha_\ast}{\alpha_\ast+\beta_\ast}
$$

$$
p(\tilde{x}=0|D)=\frac{B(\alpha_\ast,\beta_\ast+1)}{B(\alpha_\ast,\beta_\ast)}=\frac{\beta_\ast}{\alpha_\ast+\beta_\ast}
$$

になる。  
従って、予測分布は

$$
p(\tilde{x}|D)=\left( \frac{\alpha_\ast}{\alpha_\ast+\beta_\ast}\right) ^{\tilde{x}}\left(\frac{\beta_\ast}{\alpha_\ast+\beta_\ast}\right)^{1-\tilde{x}}
$$

となり、確率$\alpha_\ast / \alpha_\ast+\beta_\ast$のベルヌーイ分布となる。

ここまでの内容をコードにて示す。

```python
#事後確率の計算
beta_post = Beta(post_params[2][0],post_params[2][1])
q = 0.3

#30%以上であるという仮説
print("事後確率による結果:",1 - beta_post.cdf(q))

#事後オッズ比の計算
odds = (1 - beta_post.cdf(q))/beta_post.cdf(q)
print("事後オッズ比による結果:",odds)

#ベイズファクターでの比較
pre_odds = (1- beta_pre.cdf(q))/beta_pre.cdf(q)
print("ベイズファクターでの比較:",np.log10(odds/pre_odds))

#SDDRの計算
#30%かどうか
sddr = np.log10(scipy.stats.beta.pdf(0.3, alpha1, beta1)/scipy.stats.beta.pdf(0.3, post_params[2][0], post_params[2][1]))
print("SDDR:",sddr)

#確率変数の予測
a_ast, b_ast = post_params[2]
print("予測した確率変数:",a_ast/(a_ast+b_ast))
```

|             項目    |       値  |
| ----|----|
|事後確率による結果 |0.0627836|
|事後オッズ比による結果 |0.06698944|
|ベイズファクターでの比較 |-2.7598357|
|SDDR|-1.0076445793821351|
|予測した確率変数 |0.25769230769230766|

事後確率は極めて0に近いため、この事後分布においては内閣支持率が30%を超える確率はかなり低いとわかる。事後オッズ比もかなり低いことから、こちらからも同様の結論が導かれる。  
ベイズ・ファクター,SDDRに関してはJeffreys(1961)に示された支持の度合いの関係を参考にする。

|             等級    |       ベイズ・ファクターの常用対数値  | $H_1$に対する支持|
| ----|:---:|----|
|0 |$0<\log_{10}({\rm B_{01}})$|$H_0$が支持される|
|1 |$-1<\log_{10}({\rm B_{01}})<0$|それほどではない|
|2 |$-1<\log_{10}({\rm B_{01}})<-\frac{1}{2}$|相当なものである|
|3 |$-\frac{3}{2}<\log_{10}({\rm B_{01}})<-1$|強い|
|4 |$-2<\log_{10}({\rm B_{01}})<-\frac{3}{2}$|かなり強い|
|5 |$\log_{10}({\rm B_{01}})<2$|決定的である|

この表から、ベイズ・ファクター等級は5であり、$H_1$の支持は決定的と言える。SDDRに関しても等級は3なので、支持が強いと言える。

予測した確率変数に関しては、データが生成されたベルヌーイ分布の真の成功確率0.25にかなり近くなっているが、この値はデータによって変わりうるので特段意味はない。