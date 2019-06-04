# CVFX-Team8-HW6

## 1. The video
* 錄製影片，上傳至 Youtube
> 點開圖片畫面即可

|方法| ORB_SLAM2 | Other Post-Production Software |
| :--------: | :--------: | :--------: | 
|影片| [![](http://img.youtube.com/vi/gWy6zM0XJOI/0.jpg)](http://www.youtube.com/watch?v=gWy6zM0XJOI "")      | [![](http://img.youtube.com/vi/25zuljElHyM/0.jpg)](http://www.youtube.com/watch?v=25zuljElHyM "")    |
|時間| 8 sec | 14 sec |
|運鏡手法| 拍攝筆電，運鏡從左至右，接者從上（螢幕）至下（鍵盤），最後離拍攝物拉遠至近拍攝 | 拍攝筆電，運鏡幾乎未更動，頂多手晃 |


---
## 2. Visual effects with ORB-SLAM2
### 1. 建置 ORB_SLAM2 環境
* [ORB_SLAM2 環境教學](https://github.com/cvfx-2019/homework6-Match-Moving-SLAM)

    - OS: linux
    - C++ 11 Compiler
    - Pangolin
    - OpenCV
    - Eigen3
    - DBoW2
    - g2o

### 2. 影片轉換成圖像 & 生成運行時需要的文本
* 將影片轉換成一幀幀的圖片

    - 八秒的影片生成 267 張圖片

* **表示圖片集的內容** 

    - rgb.txt
    - 內容為 Timestamp、檔名
    - 生成範例如下：
```
# colour images
#file: 'echo'
# timestamp filename
0.000000 rgb/img0001.png
.033333 rgb/img0002.png
.066666 rgb/img0003.png
.099999 rgb/img0004.png
.133332 rgb/img0005.png
.166665 rgb/img0006.png
.199998 rgb/img0007.png
.233331 rgb/img0008.png
...
```
* [相關教學](https://github.com/raulmur/ORB_SLAM2/issues/486?fbclid=IwAR0bdntViLgx1YlJ2x9FrEwb5mE_R9HH6L1ikugzJVk8hl67Ty11lLS4Mkw)

### 3. 運行 ORB_SLAM2
* 運行時，影片對應的SLAM

| 圖片 | ![](https://i.imgur.com/3Dc3krs.png)| ![](https://i.imgur.com/cYzxNX1.png)|
| :--------: | :--------: | :--------: |
| 意義     | 俯視圖     | 側視圖，看出來有抓到3D的結構    |

* **得到攝影機移動時的座標相對位置**

    * KeyFrameTrajectory.txt
    * 內容為 Timestamp、x、y、z、q_x、q_y、q_z、q_w

    * 生成範例如下：
```
0.000000 0.0000000 0.0000000 0.0000000 0.0000000 0.0000000 0.0000000 1.0000000
0.433329 0.0920812 -0.0011300 0.0135294 0.0002372 -0.0082569 0.0039580 0.9999580
0.466662 0.0919333 -0.0010572 0.0129693 0.0005321 -0.0029605 0.0052868 0.9999815
0.899991 0.0972586 -0.0019400 -0.0089078 0.0017996 0.0633153 0.0143011 0.9978895
...
```

### 4. 插入物件
* 根據攝影機移動的座標位置資料來適當插入物件
* 使用兩種方法實作

| 插入物件 | ![](https://i.imgur.com/9iUkwFm.png)| ![](https://i.imgur.com/mlKBknl.png)|
| :--------: | :--------: | :--------: |
| 工具    | Python     | Unity     |
| 結果    | [![](http://img.youtube.com/vi/GM3poc79PtM/0.jpg)](http://www.youtube.com/watch?v=GM3poc79PtM "")     | [![](http://img.youtube.com/vi/Q1jj5DnL8so/0.jpg)](http://www.youtube.com/watch?v=Q1jj5DnL8so "")    |

### 5-1.討論 Python 法
* 作法：

    - 每次到達新的keyFrame會計算離下個keyframe間的變化量
    - 每次frame會根據之前計算的keyFrame間變化量更新x, y
    - 以x-axis為例，假設下一個keyFrame是0.5s後, 那麼x -=scale*(x2-x1)/fps/0.5，直到達到下一個keyFrame
       p.s. 由於(x2-x1)是相機移動的方向，因此在圖片中應該是要往反方向進行
    - scale越大，移動的速度越快
    - 會根據新的x跟y，在每個frame插入一個保齡球的圖片(2D)
    - 最後用程式把圖片們合成影片
* 部分演算法code：
```
for filename in lst:
    img = cv2.imread('./rgb/'+filename)
    img = cv2.cvtColor(img, cv2.COLOR_RGB2RGBA)
    if name2key('rgb/'+filename)==nextKeyFrame: #reach next key frame
        key += 1
        nextKeyFrame = keyFrame["time"][key] #update next key frame
        deltaTime = keyFrame["time"][key] - keyFrame["time"][key-1]
        xd = int((keyFrame['x'][key]-keyFrame['x'][key-1]) * scale_x /30 / deltaTime)
        yd = int((keyFrame['y'][key]-keyFrame['y'][key-1]) * scale_y /30 / deltaTime)
    
    x -= xd
    y -= yd
    
    #insert bowling ball base on x, y
    ......
```
* **前期看起來很正常，鏡頭往右的時候，保齡球一樣留在左邊。但後期卻整個亂掉**

* 影片結果顯示物件很容易偏掉，可能原因如下：
    
    - Scale 參數

        - 根據 KeyFrameTrajectory 得出的相對座標資料，在 pixel 計算上沒有一定的公式可得出，必須透過不斷的實驗，才能 tune 好 scale
    - ORB_SLAM2 沒做好，也沒考慮到鏡頭參數的調整
    - 沒有考慮z-axis and Quaternion

### 5-2.討論 Unity 法
* 作法：
    - 把原始影片插在Unity背景裡，不會隨camera角度、位置移動而有變化
    - Insert a 3D object, Shpere
    - 然後寫script根據trajectory file使Unity camera的x、y、z軸和Quaternion角度隨時間連續性移動(Lerp)，藉此模擬真實鏡頭的位置和角度。[相關教學](https://stackoverflow.com/questions/36850253/move-gameobject-over-time)
      
* 跟上個方法相比，效果略好點，但中間的時候還是略有跑掉
* 偏掉的原因可能是trajectory的資料可能沒那麼準確
* unity camera的視野與我們實際上拍攝影片的鏡頭有差異


---
## 3. Visual effects with any post-production software
* **使用軟體：MAKAR**
* MAKAR是一個創建AR/VR編輯平台，具有簡單的編輯介面及內建模組功能
### 1.先拍鍵盤作為實境中的辨識圖
* 其中發現所選擇的辨識圖要越複雜辨識出來的效果越好

![](https://imgur.com/j6iZzwY.jpg)

### 2.在場景中建立 3D model
* 我們插入一顆黃色球體

### 3. 與裝置做連結
* 開啟鏡頭搜尋辨識圖，讓黃色球體在現實場景中有類似保齡球的效果

[![](http://img.youtube.com/vi/CFwSNMSKeWw/0.jpg)](http://www.youtube.com/watch?v=CFwSNMSKeWw "")

* 共計 15 sec
* 運鏡手法

    - 靠近球體
    - 拉遠球體
    - 俯視


---
## 4. Compare above methods

| 方法 | ORB_SLAM2_Python | ORB_SLAM2_Unity |SLAM_MAKAR|
| :--------: | :--------: | :--------: | :--------: |
| 影片     | [![](http://img.youtube.com/vi/GM3poc79PtM/0.jpg)](http://www.youtube.com/watch?v=GM3poc79PtM "")     | [![](http://img.youtube.com/vi/Q1jj5DnL8so/0.jpg)](http://www.youtube.com/watch?v=Q1jj5DnL8so "")      | [![](http://img.youtube.com/vi/CFwSNMSKeWw/0.jpg)](http://www.youtube.com/watch?v=CFwSNMSKeWw "")     |

* ORB_SLAM2_Python
    
    - 優點
        - 實作方法簡單、直觀，Python也好寫
        - 需要調整的參數只有一個而且算直觀
        - 不用下載很肥的Unity或後製軟體，即可簡單達到所需效果
    - 缺點

        - 從「座標軸移動的位置」對應到圖片中的pixel沒有既定公式（需肉眼評量）
        - 需要 tune scale
* ORB_SLAM2_Unity
    
    - 優點
    
        - 有虛擬相機調整xyz軸的相對變動，概念上比較簡單
        - 在插入物件時，使用上比 Python 找 Pixel 直觀
    - 缺點
        - 實際拍攝所用的鏡頭與Unity camera並不完全相同，要調整有難度、不直覺
        - 需要上手此軟體的操作、C#沒python方便
        - 需要好的硬體執行，才不容易當機或卡卡的

* SLAM_MAKAR
    
    - 優點
    
        - 表現效果最好
        - 3D 球體的確有隨者攝影距離，物體表現出適當的放大縮小跟角度觀看效果
    - 缺點

        - 需要上手此軟體的操作

* **表現：MAKAR > Unity > Python**


---
## 5. Make some special effects
* 根據拍攝手法不同，物件的放大/縮小/角度效果

[![](http://img.youtube.com/vi/CFwSNMSKeWw/0.jpg)](http://www.youtube.com/watch?v=CFwSNMSKeWw "")

* 3D 球體的確有隨者攝影距離，做到適當的放大縮小跟角度觀看效果


---

