# 以時間維度分析BTC波動度
2024/01
本報告呈現如何使用幣安交易所API取得價格資料，以時間維度針對多個標的波動度與成交量進行分析，最後以視覺化圖表展示分析結果並建構投資組合。

內容分為三部分：
1. 下載資料與資料前處理
2. 計算與分析結論
3. 投資組合
4. Appendix

## 下載資料與資料前處理
### 資料欄位說明
使用幣安交易所API取得各幣種永續合約15分k價格資料（2021/01/01 ~ 即時資料）  
資料經過前處理後，格式如下：

|  columns   | unit | meaning |
|  ----  | ----  | ---- |
| open_time  | datetime | 開盤時間
| open  | float | 開盤價格
|  high | float | 最高價
|  low | float | 最低價
| close  | float | 收盤價（若還未收盤則是現價）
| volume  | float | 成交量（Symbol）
| close_time  | datetime | 收盤時間
|  #trade | int | 成交筆數
|  taker_buy_vol | float | 主動買入成交量（Symbol）
|  week | int | 星期幾
|  time_zone | str | 世界各地區盤中時間（後有詳述）

### 交易時段說明
將交易時段分成以下幾種，重疊部分只記一次。

|  市場   | 時段（UTC）| 代碼
|  ----  | ----  |  ----  | 
|亞盤 |23:00 - 6:00|AS|
|歐亞重疊 |06:00 - 08:00|ASEU|
|歐盤 |08:00 - 12:00|EU|
|歐美重疊 |12:00 - 15:00|EUNA|
|美盤| 15:00 - 21:00|NA|
|其他| 21:00 - 23:00 | OTHERS

### 資料遺失問題

- 圖一，2021-05-19 13:30:00 BNB （漲29%） ![Alt text](image-1.png)
- 圖二，2023-01-02 06:45:00	SOL （漲20%） ![Alt text](image-2.png)

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>open_time</th>
      <th>open</th>
      <th>high</th>
      <th>low</th>
      <th>close</th>
      <th>volume</th>
      <th>close_time</th>
      <th>#trade</th>
      <th>taker_buy_vol</th>
      <th>return</th>
      <th>week</th>
      <th>time_zone</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2021-01-01 00:00:00</td>
      <td>28948.19</td>
      <td>29045.93</td>
      <td>28706.00</td>
      <td>28786.75</td>
      <td>3181.960</td>
      <td>2021-01-01 00:14:59.999</td>
      <td>24166.0</td>
      <td>1277.389</td>
      <td>-0.557686</td>
      <td>5</td>
      <td>AS</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021-01-01 00:15:00</td>
      <td>28786.92</td>
      <td>28902.68</td>
      <td>28752.30</td>
      <td>28859.28</td>
      <td>2215.315</td>
      <td>2021-01-01 00:29:59.999</td>
      <td>17044.0</td>
      <td>1209.372</td>
      <td>0.251956</td>
      <td>5</td>
      <td>AS</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2021-01-01 00:30:00</td>
      <td>28860.00</td>
      <td>28968.49</td>
      <td>28859.99</td>
      <td>28947.61</td>
      <td>1153.678</td>
      <td>2021-01-01 00:44:59.999</td>
      <td>12208.0</td>
      <td>620.757</td>
      <td>0.306071</td>
      <td>5</td>
      <td>AS</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2021-01-01 00:45:00</td>
      <td>28947.61</td>
      <td>29055.00</td>
      <td>28910.94</td>
      <td>29015.00</td>
      <td>1486.635</td>
      <td>2021-01-01 00:59:59.999</td>
      <td>12524.0</td>
      <td>876.324</td>
      <td>0.232800</td>
      <td>5</td>
      <td>AS</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2021-01-01 01:00:00</td>
      <td>29015.00</td>
      <td>29499.00</td>
      <td>28975.46</td>
      <td>29446.35</td>
      <td>7476.067</td>
      <td>2021-01-01 01:14:59.999</td>
      <td>45858.0</td>
      <td>4619.428</td>
      <td>1.486645</td>
      <td>5</td>
      <td>AS</td>
    </tr>
  </tbody>
</table>
</div>

## 計算與分析

### 波動度
使用標準差公式
$$
\begin{aligned}
    \sigma = \sqrt{\frac{1}{n-1}\sum_{i=1}^{n} (x_i - \bar{x})^2}
\end{aligned}
$$
分別計算以下幾種不同情況的波動度：
|  title   |  meaning |
|  ----  | ---- 
| 15m_alltime  | 資料時間內15分K歷史波動度
| 1h_alltime  |  1H歷史波動度
|  1d_alltime | 1日歷史波動度
|week_day| 分別計算一週內不同天的波動度
|time_zone| 針對不同市場活躍時間計算1H波動度

#### BTCUSDT分析圖表
##### 週間分析
- 圖三，週間日平均波動   
  ![週間日波動](image-3.png)  
- 圖四，週間15m報酬分布   
    ![週間15m報酬分布](image-6.png)  
  
    ~~從圖三可看出，週五的波動度明顯較其他平日更高，推測是因為週期貨與週選擇權的交割導致價格波動的產生，也因此，週五的成交量相較其他天並無明顯增加。~~  

    週六週日的波動度與成交量明顯低於平日，推測是因為各國期貨外匯市場假日休盤，導致**市場在假日時較缺乏流動性**。以盒鬚圖（Box plot）觀察15m級別價格波動的分布，平日的15m波動平均在0.6%左右，假日則是低於0.5%。

---  

- 圖五，週間15m平均成交量  
  ![週間15m平均成交量](image-4.png)  
  由圖五可看出，除了波動以外，假日的成交量也明顯不及平日，推測原因與上述相同。  

- 圖六，週間15m單筆平均成交量  
  ![週間15m單筆平均成交量](image-5.png)  
  有趣的是，將圖五與圖六一起分析，會發現單筆平均成交量的差異相對小很多，這表示假日的交易以小額交易為主（散戶），因此將平均過後單筆成交量的差距才會變小。側面應證了市場主力在平日交易居多。  

---

##### 比較日內不同市場活躍時段

|  市場   | 時段（UTC）| 代碼
|  ----  | ----  |  ----  | 
|亞盤 |23:00 - 6:00|AS|
|歐亞重疊 |06:00 - 08:00|ASEU|
|歐盤 |08:00 - 12:00|EU|
|歐美重疊 |12:00 - 15:00|EUNA|
|美盤| 15:00 - 21:00|NA|
|其他| 21:00 - 23:00 | OTHERS

- 圖七，不同時區1H平均波動  
  ![不同時區1H波動](image-7.png)   
  
- 圖八，不同時區15m平均成交量  
  ![不同時區15m平均成交量](image-8.png)  
  
- 圖九，不同時區15m平均單筆成交量  
  ![不同時區15m平均單筆成交量](image-9.png)  
    
  上圖七可觀察到，亞歐盤重疊的時候，波動度明顯較低；歐美盤重疊時，波動率明顯較高  
  
    上圖八可觀察到，主要的成交量都產生在歐美盤重疊與美盤時段。
    綜合這兩張圖一起看會發現，亞盤與亞歐盤重疊的這兩個時段中，雖然**成交量差不多，但波動度卻是單獨亞盤的時候較高**，下一個區段會揭示離群值的資料，我認為可以解釋這個現象。
    
    在歐美盤重疊與美盤時，成交量明顯大於其他時段，由此可推論，**美盤資金在比特幣市場的影響力是相對較大的**，且可以發現歐美盤重疊時波動率極高，我認為是美盤開盤所導致，這也可以**反向證明為何亞盤的成交量這麼低**，因為對美盤的交易者來說，亞盤時間正好是睡覺時間。

---

- 圖十，不同時區15m平均報酬分布  
  ![不同時區15m報酬分布](image-10.png)  

- 圖十一，不同時區15m平均報酬分布(離群值)  
  ![Alt text](image-14.png)  
  上圖十與圖十一可觀察到，歐美盤重疊（美盤剛開盤）與美盤時，在小級別的平均波動明顯比其他時段要大。
    也再一次應證前面所述，**我認為目前比特幣市場的主要資金與美盤的資金有較高度的重疊**。

---

- 圖十二，週間不同時區平均波動比  
  ![週間不同時區平均波動比較](image-11.png)  

---
### 分析結論

本報告中，針對兩種時間維度分析比特幣的波動度與成交量變化：
  
- 比較週間每日的波動度與成交量
- 比較日內不同市場活躍時段
  
以下是我的發現：
- 比特幣市場在平日（週一～週五）是相對活躍的。
- 根據成交量，可以推測出比特幣市場在假日時主要以散戶小打小鬧為主。
- 透過主動買入成交量的數據，也應證機構大戶主要以Maker的角色進場。
- 綜合波動度與成交量，推斷出比特幣市場目前以美盤資金為最大宗，市場波動時間對亞洲交易者較為不友善。

---

## 投資組合
我建構了兩種投資組合與完全Equal weight做比較。
此部分使用 QuantStats 套件輔助分析

1. 波動分配組合
   以各個幣種的平均日波動率作為資金分配依據，波動大的分配給多一點的資金。
   
   ||Equal|波動分配|
   |---|:---:|:---:|
   |Cum. Return|872.82%|1,236.44%|
   |MDD|-79.05%|-81.75%|
   |Sharpe|1.1|1.18|
   |Sortino|1.63|1.76|
   |Calmar|0.84|0.97|
   |Profir Factor|1.22|1.24|

   ![Alt text](image-56.png)
2. 逆勢操作組合
   根據前一天日線漲跌，逆勢進場。
    ||波動分配|逆勢操作
   |---|:---:|:---:|
   |Cum. Return|1,236.44%|1,606.81%
   |MDD|-81.75%|-70.82%|
   |Sharpe|1.18|1.32
   |Sortino|1.76|2.15
   |Calmar|0.97|1.26|
   |Profir Factor|1.24|1.35

    ![Alt text](image-57.png)



---
## Appendix

### ETHUSDT 分析圖表
##### 週間分析
![Alt text](image-15.png)  
![Alt text](image-16.png)  
![Alt text](image-17.png)  
![Alt text](image-18.png)  
##### 比較日內不同市場活躍時段
![Alt text](image-19.png)  
![Alt text](image-20.png)  
![Alt text](image-21.png)  
![Alt text](image-22.png)  
![Alt text](image-23.png)  
---
### XRPUSDT 分析圖表
##### 週間分析
![Alt text](image-24.png)  
![Alt text](image-25.png)  
![Alt text](image-26.png)  
![Alt text](image-27.png)  
![Alt text](image-28.png)  
##### 比較日內不同市場活躍時段
![Alt text](image-31.png)  
![Alt text](image-32.png)  
![Alt text](image-33.png)  
![Alt text](image-34.png)  
![Alt text](image-35.png)  
---
### BNBUSDT 分析圖表
##### 週間分析
![Alt text](image-36.png)
![Alt text](image-37.png)
![Alt text](image-38.png)
![Alt text](image-39.png)
![Alt text](image-40.png)
##### 比較日內不同市場活躍時段
![Alt text](image-41.png)
![Alt text](image-42.png)
![Alt text](image-43.png)
![Alt text](image-44.png)
![Alt text](image-45.png)

### SOLUSDT 分析圖表
##### 週間分析
![Alt text](image-46.png)
![Alt text](image-47.png)
![Alt text](image-48.png)
![Alt text](image-49.png)
![Alt text](image-50.png)
##### 比較日內不同市場活躍時段
![Alt text](image-51.png)
![Alt text](image-52.png)
![Alt text](image-53.png)
![Alt text](image-54.png)
![Alt text](image-55.png)