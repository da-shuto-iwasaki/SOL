# SOL
電通SOL ルールを加味した制度検証

## ルールベース
### 下準備
1. `df_brand` を用いて各ブランドごとに目標となる「週ごとのTRP」を記録する。
2. 「発行期間」や「禁止時間」もブランドごとに記録する。
3. ターゲット階層が同じでCMの秒数も同じブランドは、ブランドグループとして１つにまとめる。（なお、ここでCMの秒数が異なる場合は争うことがないので別にする。）
4. ブランドグループの「週ごとの目標TRP」も記録する。
5. `df_cm` からブランドグループの「ターゲット階層の視聴率」、「時間帯」、「日にち」のみを抜き出す。
6. 各CM枠に関して、「二番目に大きな値」で割った比率を求める。

### 利用するデータ構造
#### `df_brand`
各ブランドの情報を記述する。

| colname  | 内容                     |
| -------- | ------------------------ |
| sec      | CM秒数                   |
| target   | 指定階層                 |
| code     | ブランドコード           |
| day_span | 発注期間                 |
| *n_w     | n番目の週に取得したいTRP |
| bad_time | 禁止時間                 |

#### `df_brand_gp`
ブランドグループの情報を記述する。
| colname    | 内容                                          |
| ---------- | --------------------------------------------- |
| sec        | CM秒数                                        |
| target     | 指定階層                                      |
| n_w_isfull | n番目の週が目標達成したか。達成したなら`True` |

#### `df_cm`
各CM枠の情報を記述する。

| colname    | 内容        |
| ---------- | ----------- |
| sec        | CM秒数      |
| day        | 放送日      |
| start_time | 開始時間    |
| end_time   | 終了時間    |
| *TRP       | 各階層のTRP |

#### `df_week`
`df_brand` の `n_w` の 'n' と 実際の日にちの対応関係。
| colname | 内容               |
| ------- | ------------------ |
| 1w      | 2017/7/30-2017/8/6 |
| 2w      | 2017/8/7-2017/8/13 |
|         |                    |

### 擬似コード
```python
cm_dict = dict()

while True:
    '''df_cm で最も大きな値をとるCM枠を取り出す。'''
    max_index = CM の index
    max_bg    = ブランドグループ
    max_trp   = TRP
    max_day   = 放送日
    day_id    = `n`w と呼ばれている番号

    '''df_brand_gp を参照し、すでに目標を達成しているかを調べる。'''
    if df_brand_gp.day_id:
        df_cm.loc[max_index,max_bg] = 0 # すでに目標を達成していれば、割合を0に書き換えて終了。
        continue

    '''df_brand を参照し、取得可能かを調べる。'''
    df_tmp = df_brand[df_brand.target == max_bg] # 指定階層のブランドを抜き出す。
    if len(df_tmp) == 0:
        df_cm.loc[max_index,max_bg] = 0 # 満たすものがなければ、割合を0に書き換えて終了。
        continue

    '''取得可能'''
    allo_code = df_tmp[max_day in df_tmp.day_span].sample().code # 発注期間内に入っているブランドからランダムチョイス。
    df_brand[df_brand.code == allo_code][day_id] -= max_trp # 週に取得したいTRPを引く。
    cm_dict[max_index] = allo_code # CMとそれを割り当てるブランドコードの対応関係を記録する。

    '''全体終了条件'''
    if df_brand_gp.n_w_isfull.all() == True or len(df_cm) == 0:
        break
```
