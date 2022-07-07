# OpenCV：把卡Ｐ上圖

## 問題

將信用卡貼在圖中，要符合原圖中卡的位置和大小

## 資料
|img.jpg|mask.jpg|new_card.jpg|
|:-:|:-:|:-:|
|![](https://i.imgur.com/qAMqcvK.jpg =70%x)|![](https://i.imgur.com/0eCn9mk.jpg =70%x)|![](https://i.imgur.com/7WRjyHY.jpg)|

## 流程
> 1. 得到mask四個角的座標
> 2. 使用透視轉換來轉換new_card to mask
> 3. 挖出img中的卡片
> 4. 貼上經過透視轉換的new_card於img上

### 1. 得到mask四個角的座標

#### 邊緣偵測 - Canny

對mask做邊緣偵測

![](https://i.imgur.com/K94f2jH.png =40%x)

#### 輪廓偵測 - Contour
- 對邊緣偵測過的圖做輪廓偵測
- 使用``cv2.findContours``偵測
    - External：若有其他輪廓包在內部的話，只取外層的輪廓
    - CHAIN_APPROX_NONE：不做水平或垂直輪廓點的壓縮，保留所有的輪廓點
- 使用``cv2.minAreaRect``取出中心點,寬高,角度，``cv2.boxPoints``去轉換成四個角的座標
- 將座標以np.float32的形式存入``dst``

|四個點|繪製出輪廓和四點|
|:-:|:-:|
|![](https://i.imgur.com/DwcvN8f.png)|![](https://i.imgur.com/R9g8Ar6.png =60%x)|![](https://i.imgur.com/dAkvRan.png)


### 2. 透視轉換

- 座標
``src``：原圖
``dst``：要轉換成的圖片

```python
dst = box
src = np.array([[0, 0], [new_card.shape[1] - 1, 0], [new_card.shape[1] - 1,
               new_card.shape[0] - 1], [0, new_card.shape[0] - 1]]).astype(np.float32)
```
- 計算轉換矩陣
```python
warp_mat = cv2.getPerspectiveTransform(src, dst)
```
- 轉換
```python
warp_dst = cv2.warpPerspective(
    new_card, warp_mat, (mask.shape[1], mask.shape[0]))
```

轉換後的圖片
![](https://i.imgur.com/Bb1evM4.png =40%x)




### 3. 挖出img中的卡片

#### 將mask黑白轉換

直接減去255``mask_blackObj = 255 - mask``

![](https://i.imgur.com/rkXfqNb.png =40%x)

#### img疊加mask

使用``cv2.bitwise_and``疊加剛剛黑白轉換的mask

![](https://i.imgur.com/4tsnYWV.jpg =40%x)

### 4. 貼上經過透視轉換的new_card於img上

使用``cv2.add``去合併

![](https://i.imgur.com/IMpt0St.jpg =40%x)


---
Reference：
[array建立](https://medium.com/@python.coding.site/numpy%E5%AD%B8%E7%BF%92101-6acb9fb3a6fe)
[affine transform(for 三角形的) python code](https://docs.opencv.org/3.4/d4/d61/tutorial_warp_affine.html)
[Perspective transform講解](https://theailearner.com/tag/cv2-getperspectivetransform/)
[影像遮罩](https://steam.oxxostudio.tw/category/python/ai/opencv-mask.html)
[contour參數講解](https://chtseng.wordpress.com/2016/12/05/opencv-contour%E8%BC%AA%E5%BB%93/)
