# d4pdfを使ってみよう!!

## d4pdfの仕様手順

### 1. 変数を決めてほしいデータを切り出そう！ - `DIASからデータをダウンロード`  
**Step1 : DIASのアカウント登録**  
[DIASアカウント登録画面](https://auth.diasjp.net/account/public/ja/guest/)から必要情報を入力しアカウントを作成  
  
**Step2 : DIASからデータをダウンロード**  
[DIASデータダウンロード](http://d4pdf.diasjp.net/d4PDF.cgi?target=GCM-subset&lang=ja)から欲しいデータをダウンロード

**SUMMARY**
>・アカウントを作成しデータをダウンロード  
>・ダウンロードの際、選択する情報は以下の5つ  
    　　1.実験（モデル）  
    　　2.期間  
    　　3.変数  
    　　4.アンサンブル  
    　　5.領域    

### 2. Pythonを使ってデータを解凍しよう！ - `ダウンロードしたデータのCSV化`
#### ・[データを解凍しよう](https://nipponkoei1946.sharepoint.com/sites/msteams_49697a/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2Fmsteams%5F49697a%2FShared%20Documents%2FGeneral%2F%E2%98%86%E5%80%8B%E4%BA%BA%E4%BD%9C%E6%A5%AD%E3%83%95%E3%82%A9%E3%83%AB%E3%83%80%2F%E6%9D%BE%E6%9C%AC%2FPython%E9%96%A2%E9%80%A3%2F%E3%82%A2%E3%83%97%E3%83%AA&viewid=9244e75a%2D67a6%2D495c%2D8f32%2D811ab6eba68c)
DIASからダウンロードしたTARファイルをPythonを使って解凍します。  
**SUMMARY**
>・ファイルの保存場所を入力  
>・ファイルの保存場所は「アドレスのテキストをコピー」から入手  

```python
import tarfile
import glob
import os
cd = input("ファイルの保存場所を入力してください")
print("Processing...")
os.chdir(cd)
path_list   = glob.glob("*.tar")
for path1 in path_list:
   path2 = path1.replace(".tar","_open")
   with tarfile.open(path1, 'r') as t:t.extractall(path2)
print("Completed")
```
#### ・CSVに変換しよう
**WARNING**
>気温と降水量ではデータの単位が異なるため同じコードでは解凍することができません。  
>そのため、気温と降水量のデータを間違えないよう注意してください。
#### ①  [CSVに変換しよう～気温編～](https://nipponkoei1946.sharepoint.com/sites/msteams_49697a/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2Fmsteams%5F49697a%2FShared%20Documents%2FGeneral%2F%E2%98%86%E5%80%8B%E4%BA%BA%E4%BD%9C%E6%A5%AD%E3%83%95%E3%82%A9%E3%83%AB%E3%83%80%2F%E6%9D%BE%E6%9C%AC%2FPython%E9%96%A2%E9%80%A3%2F%E3%82%A2%E3%83%97%E3%83%AA&viewid=9244e75a%2D67a6%2D495c%2D8f32%2D811ab6eba68c)

**SUMMARY**
>・ファイルの保存場所を入力してEnter  
>・DATファイルからグリッドの数（xとyの値）を確認し入力  

```python
import csv
import struct
import glob
import os
cd = input("ファイルの保存場所を入力してください")
os.chdir(cd)
print("Processing...")

#datファイルを結合
data_list   = glob.glob("*.dat")
with open("tem.dat", 'wb') as file:
    for data in data_list:
        with open(data, 'rb') as fin:
            file.write(fin.read())


f=open ("tem.dat",'rb')
g=open ("average_day_tem.csv",'w',newline='')
writer=csv.writer(g)

x=input ("xの列数=")
y=input ("yの行数=")

x=int(x)
y=int(y)
ctl=""
file_eof=""
counter=0
hour=0
day=0
kosi=x*y
tem_d=0.0

for t in range(100*365*kosi):
  file_eof = f.read(4)
  counter=counter+1
  if not file_eof:
    break

counter=counter-1
day=int(counter/(kosi))
print('End Of File ',day,"Days")
 
ctl="Days="+str(day)
writer.writerow([ctl])
ctl="Y="+str(y)
writer.writerow([ctl])
ctl="X="+str(x)
writer.writerow([ctl])

f.close()
f=open ("tem.dat",'rb')

tem_d=[0.0]*kosi

for t in range (day):
  for i in range(kosi):
    data=f.read(4)
    if not data: 
      break
    tem_tpl=struct.unpack(">f",data)
    tem_d[i]=tem_tpl[0]-273 #Kを℃に変換
  print(tem_d)
  writer.writerow(tem_d)

print("end")

f.close()
g.close()
```

#### ② [CSVに変換しよう～降雨編～](https://nipponkoei1946.sharepoint.com/sites/msteams_49697a/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2Fmsteams%5F49697a%2FShared%20Documents%2FGeneral%2F%E2%98%86%E5%80%8B%E4%BA%BA%E4%BD%9C%E6%A5%AD%E3%83%95%E3%82%A9%E3%83%AB%E3%83%80%2F%E6%9D%BE%E6%9C%AC%2FPython%E9%96%A2%E9%80%A3%2F%E3%82%A2%E3%83%97%E3%83%AA&viewid=9244e75a%2D67a6%2D495c%2D8f32%2D811ab6eba68c)
**SUMMARY**
>・ファイルの保存場所を入力  
>・DATファイルからグリッドの数（xとyの値）を確認し入力  
```python
import csv
import struct
import glob
import os
cd = input("ファイルの保存場所を入力してください")
os.chdir(cd)
print("Processing...")

#datファイルを結合
data_list   = glob.glob("*.dat")
with open("pre_day.dat", 'wb') as file:
    for data in data_list:
        with open(data, 'rb') as fin:
            file.write(fin.read())
f=open ("pre_day.dat",'rb')
g=open ("total_day_pre.csv",'w',newline='')
writer=csv.writer(g)

x=input ("xの列数=")
y=input ("yの行数=")

x=int(x)
y=int(y)
ctl=""
file_eof=""
counter=0
hour=0
day=0
kosi=x*y
rain_d=0.0

for t in range(100*365*kosi):
  file_eof = f.read(4)
  counter=counter+1
  if not file_eof:
    break

counter=counter-1
day=int(counter/(kosi))
print('End Of File ',day,"Days")
 
ctl="Days="+str(day)
writer.writerow([ctl])
ctl="Y="+str(y)
writer.writerow([ctl])
ctl="X="+str(x)
writer.writerow([ctl])

f.close()
f=open ("pre_day.dat",'rb')

rain_d=[0.0]*kosi

for t in range (day):
  for i in range(kosi):
    data=f.read(4)
    if not data: 
      break
    rain_tpl=struct.unpack(">f",data)
    rain_d[i]=rain_tpl[0]*3600*24 #kg/m2/sをmm/dayに変換
  print(rain_d)
  print(type(rain_d))
  writer.writerow(rain_d)

print("end")

f.close()
g.close()
```


**WARNING**
>CSVファイルの名前は「モデル名_アンサンブル数_**日平均気温**」のような形にしてください！  
>データ分析の際に使用するコードで、平均気温・最高気温・最低気温を参照して計算している部分があります。


### 3. データ分析をしよう！ - `CSVからのデータ抽出、まとめ`
#### ・[データ分析をしよう](https://nipponkoei1946.sharepoint.com/sites/msteams_49697a/Shared%20Documents/Forms/AllItems.aspx?id=%2Fsites%2Fmsteams%5F49697a%2FShared%20Documents%2FGeneral%2F%E2%98%86%E5%80%8B%E4%BA%BA%E4%BD%9C%E6%A5%AD%E3%83%95%E3%82%A9%E3%83%AB%E3%83%80%2F%E6%9D%BE%E6%9C%AC%2FPython%E9%96%A2%E9%80%A3%2F%E3%82%A2%E3%83%97%E3%83%AA&viewid=9244e75a%2D67a6%2D495c%2D8f32%2D811ab6eba68c)

**SUMMARY**
>・ファイルの保存場所を入力  
>・プロジェクトサイト（lao or cam）を確認し入力  
>・開始年と終了年をそれぞれ入力  
>・抽出したいグリッド番号を入力  
```python
import numpy as np
import pandas as pd 
import glob
import os
cd = input("ファイルの保存場所を入力してください")
os.chdir(cd)

#場所
country_list = {"cam" : [10.9516, 104.625], "lao" : [16.55, 104.65]}
place = country_list[input("場所を入力してください（lao or cam）")]

#期間の設定
y_start = int(input("開始年(yyyy)を入力してください。"))
d_start = str(y_start) + "/01/01"
y_end = int(input("終了年(yyyy)を入力してください。"))
d_end = str(y_end) + "/12/31"
days = pd.date_range(start = d_start, end = d_end)
period = y_end - y_start + 1

#グリッド番号の指定
n_grid = int(input("抽出したいグリッド番号を入力してください。"))

print("Processing...")

fileName1 = "データまとめ_モデル名_国名.xlsx"
fileName2 = "モデル名_国名.xlsx"
writer = pd.ExcelWriter(fileName1)

#CSVファイルからのデータ抽出
y_index = list(range(y_start, y_end + 1, 1))
m_index = list(range(1, 12 + 1, 1))
file_list   = glob.glob("*.csv")
for file in file_list:
    sheet = file.replace(".csv","")
    with open(file, "r") as f:
        col_names = ['c{0:02d}'.format(i) for i in range(100)]
        lst = pd.read_csv(f,delimiter=",",names = col_names,dtype='unicode')
        data = lst.iloc[3:, (n_grid - 1)].astype(float).to_list()
        raw_data = pd.DataFrame(data, index = days, columns = ["Value"])

        A = y_start
        m_sum = []
        m_ave = []
        m_max = []
        m_min = []
        ave_m = []
        y_ave = []
        y_max = []
        y_min = []
        cal_a = []
        con_r = []
        con_d = []
        abn_r = []
        abn_t = []
        max_ave = []
        min_ave = []

        #百分位数の計算
        percentile_list = [.25, .5, .75, .95]
        use_cols = ["min", "25%","50%", "75%","95%", "max"]
        df_per =  raw_data.describe(percentile_list).loc[use_cols,:]

       #月ごとの値の計算
        change_data = raw_data.set_index([raw_data.index.year, raw_data.index.month, raw_data.index])
        change_data.index.names = ['Year', 'Month', 'date']
        ave_mon = change_data.groupby(level=('Year', 'Month')).mean()
        max_mon = change_data.groupby(level=('Year', 'Month')).max()
        min_mon = change_data.groupby(level=('Year', 'Month')).min()
        sum_mon = change_data.groupby(level=('Year', 'Month')).sum()
        mon_ave = change_data.groupby(level=('Month')).mean()
        sum_ave = change_data.groupby(level=('Month')).sum() /period

        #連続降雨日数、連続干天日数用データ 
        bool1 = (raw_data >= 1)
        bool1 = (bool1["Value"].to_list())
        for i in range(len(bool1) - 1):
            if bool1[i+1] == 1:
                bool1[i+1] += bool1[i]
        rain_data = pd.DataFrame(bool1, index = days, columns = ["Value"])
        bool2 = (raw_data < 1)
        bool2 = (bool2["Value"].to_list())
        for i in range(len(bool2) - 1):
            if bool2[i+1] == 1:
                bool2[i+1] += bool2[i]
        drought_data = pd.DataFrame(bool2, index = days, columns = ["Value"])

        while A < y_end + 1:
            #値をきれいに並び変える
            m_ave.append(ave_mon.loc[A, "Value"].to_list())           
            m_max.append(max_mon.loc[A, "Value"].to_list())
            m_min.append(min_mon.loc[A, "Value"].to_list())                
            m_sum.append(sum_mon.loc[A, "Value"].to_list())

            #年ごとの値の計算
            x = raw_data[raw_data.index.year == A]
            ave_y = float(x.mean())
            max_y = float(x.max())
            min_y = float(x.min())
            y_ave.append(ave_y)
            y_max.append(max_y)
            y_min.append(min_y)
            
            #極端現象の計算
            if min_y == 0:
                #連続降雨日数
                y = rain_data[rain_data.index.year == A] 
                y = (y["Value"].to_list())
                continuous_rain = max(y)
                con_r.append(continuous_rain)
                #連続干天日数
                z = drought_data[drought_data.index.year == A]
                z = (z["Value"].to_list())
                continuous_drought = max(z)
                con_d.append(continuous_drought)
                #異常降雨日数(30mm以上)
                bool3 = (x >= 30)
                abonormal_rain = bool3.sum()
                abn_r.append(abonormal_rain)
            else:
                #異常高温日数(35℃以上)             
                bool4 = (x >= 35)
                abnormal_temperature = bool4.sum()
                abn_t.append(abnormal_temperature)
            A = A + 1
        
        #月ごとの最高最低の平均
        t = 0
        while t < 12:
            max_ave.append(sum(np.array(m_max)[:,t])/period)
            min_ave.append(sum(np.array(m_min)[:,t])/period)
            t = t + 1        

        #全体の平均・最高の平均・最低の平均の計算
        cal_a.append(sum(y_ave)/period)
        cal_a.append(sum(y_max)/period)
        cal_a.append(sum(y_min)/period)

        cr = float(sum(con_r[1:])/(period - 1))
        cd = float(sum(con_d[1:])/(period - 1))
        ar = float(sum(abn_r)/period)
        at = float(sum(abn_t)/period)

        #データフレームの作成
        df_m_ave = pd.DataFrame(m_ave, columns=pd.Index(m_index),index = pd.Index(y_index))             
        df_m_max = pd.DataFrame(m_max, columns=pd.Index(m_index),index = pd.Index(y_index)) 
        df_m_min = pd.DataFrame(m_min, columns=pd.Index(m_index),index = pd.Index(y_index))
        df_m_sum = pd.DataFrame(m_sum, columns=pd.Index(m_index),index = pd.Index(y_index))
        df_y = pd.DataFrame([y_ave, y_max, y_min, y_index], index = ["Ave", "Max", "Min", "Year"])
        df_y =  df_y.T.set_index("Year")
        df_max_ave = pd.DataFrame(max_ave)
        df_min_ave = pd.DataFrame(min_ave)
        df_cal  = pd.DataFrame(cal_a)
        df_place = pd.DataFrame(place)
        df_ext_w = pd.DataFrame([cr, cd, ar, at], index = ["Continuous Rain", "Continuous Drought", "Abnormal Rain", "Abnormal Temperature"])

        #データの書き込み
        pd.DataFrame(["Ave"]).to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=2, startcol=9)
        pd.DataFrame(["Max"]).to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=69, startcol=9)
        pd.DataFrame(["Min"]).to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=136, startcol=9)
        pd.DataFrame(["Sum"]).to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=203, startcol=9)
        raw_data.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=2, startcol=1)
        df_y.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=2, startcol=4)
        df_place.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=1, startcol=1)
        df_cal.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=1, startcol=5)
        df_per.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=69, startcol=4)
        df_m_ave.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=2, startcol=9)
        df_m_max.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=69, startcol=9)    
        df_m_min.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=136, startcol=9)
        df_m_sum.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=203, startcol=9)
        mon_ave.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=1, startcol=10)
        df_max_ave.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=68, startcol=10)        
        df_min_ave.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=135, startcol=10)
        sum_ave.T.to_excel(writer, sheet_name=sheet,index=False, header=False, startrow=202, startcol=10)
        df_ext_w.to_excel(writer, sheet_name=sheet,index=True, header=True, startrow=78, startcol=4)

writer.save()

#データまとめシートの作成
writer2 = pd.ExcelWriter(fileName2)
prof_data = place
prof_data.append(n_grid)
summary_table1 = []
summary_table2 = []
summary_table3 = []
index_st1 = ["日平均気温", "日最高気温", "日最低気温", "日最大降水量", "日最少降水量", "連続降雨日数", "連続干天日数", "異常高温日数", "異常降水日数"]
index_st2 = ["月別平均気温", "月別最高気温", "月別最低気温"]
index_st3 = ["月別降水量"]
df_sheet_all = pd.read_excel('データまとめ_モデル名_国名.xlsx', sheet_name=None, index_col=0)
for i in df_sheet_all.keys():
    if "平均気温" in i:
        TAve = i
    elif "最高気温" in i:
        TMax = i
    elif "最低気温" in i:
        TMin = i
    else:
        Pre = i
df_Tave = df_sheet_all[TAve]
df_TMax = df_sheet_all[TMax]
df_TMin = df_sheet_all[TMin]
df_Pre = df_sheet_all[Pre]
summary_table1.append(df_Tave.iloc[0, 4])
summary_table1.append(df_TMax.iloc[0, 4])
summary_table1.append(df_TMin.iloc[0, 4])
summary_table1.append(df_Pre.iloc[0, 5])
summary_table1.append(df_Pre.iloc[0, 6])
summary_table1.append(df_Pre.iloc[78, 4])
summary_table1.append(df_Pre.iloc[79, 4])
summary_table1.append(df_TMax.iloc[81, 4])
summary_table1.append(df_Pre.iloc[80, 4])
summary_table2.append(df_Tave.iloc[0, 9:21].to_list())
summary_table2.append(df_TMax.iloc[67, 9:21].to_list())
summary_table2.append(df_TMin.iloc[134, 9:21].to_list())
summary_table3.append(df_Pre.iloc[201, 9:21].to_list())
df_summary1 = pd.DataFrame(summary_table1, index = pd.Index(index_st1))
df_summary2 = pd.DataFrame(summary_table2, index = pd.Index(index_st2), columns = pd.Index(m_index))
df_summary3 = pd.DataFrame(summary_table3, index = pd.Index(index_st3), columns = pd.Index(m_index))
df_prof_data = pd.DataFrame(prof_data, index = ["緯度", "経度", "グリッド番号"])
df_summary1.to_excel(writer2, sheet_name="まとめ",index=True, header=False, startrow=1, startcol=1)
df_summary2.to_excel(writer2, sheet_name="まとめ",index=True, header=True, startrow=1, startcol=4)
df_summary3.to_excel(writer2, sheet_name="まとめ",index=True, header=True, startrow=6, startcol=4)
df_prof_data.to_excel(writer2, sheet_name="まとめ",index=True, header=False, startrow=11, startcol=1)
writer2.save()
print("Completed")
```
