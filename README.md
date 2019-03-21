# Homework2-style-transfer
* Dataset: ukiyoe2photo
* Original model: MUNIT
* Other method: FastPhotoStyle

## Training MUNIT
我們選擇的dataset為ukiyoe2photo，trainA為浮世繪，trainB為真實風景照  

下圖為MUNIT在training中所存的.pth檔截圖：

![](https://i.imgur.com/IUaUTq4.jpg) ![](https://i.imgur.com/hCiK3LQ.jpg) 
  
下圖為training過程中跑85萬筆iterations的結果：
- 浮世繪→風景照

![](https://i.imgur.com/byvjs3t.jpg)

- 風景照→浮世繪

![](https://i.imgur.com/thVCVwN.jpg)
 
## Inference MUNIT
生成的部分，我們以`ukiyoe`與`風景照`作為關鍵字來搜尋，並各取10張圖片，model則使用85萬筆iterations的結果來跑生成

以下為MUNIT的生成結果：

- 浮世繪→風景照（左圖為input，右為不同風格的生成結果）

![](https://i.imgur.com/onENtYc.png)
![](https://i.imgur.com/7oe9dqN.png)
![](https://i.imgur.com/s4v7nCa.png)
![](https://i.imgur.com/kTvJOjg.png)
![](https://i.imgur.com/eXPTzt7.png)
![](https://i.imgur.com/ePkWzfV.png)
![](https://i.imgur.com/oJhWQjB.png)
![](https://i.imgur.com/Fu8zu14.png)
![](https://i.imgur.com/rFtmwUq.png)
![](https://i.imgur.com/G9amjRv.png)

- 風景照→浮世繪（左圖為input，右為不同風格的生成結果）

![](https://imgur.com/JDUfIiu.png)

### Analysis

依此結果來看，生成後的圖像比預期的還不理想許多：
* 以浮世繪轉風景照而言，由於浮世繪的畫風及用色方式與現實景物有所差異，model可能較無法辨認浮世繪圖內的物體或景色為何，導致生成的圖內景物與原圖相距甚遠，甚至失去圖片該有的輪廓，變成一團團的色塊。且MUNIT轉換後的材質都是自然風景的元素，所以在轉換「建築物」或「人物」時，output會更加脫離原本浮世繪的構圖（不過這可能是因為風景照的dataset中本身就缺乏人物以及建築物的照片，使得生成結果皆以自然風景為主）。
* 由風景照轉浮世繪的結果能看出，model較能抓取到輪廓明顯的物體或較光亮的景色，但生成的結果還是沒能保留住原圖應有的特色，變成一塊塊不規則狀的形體。以圖像的材質來說，有大致表現出與浮世繪較為相近的質地與色調，但10種不同風格的圖片中，除了背景的顏色不太一樣以外，物體所呈現的色彩大致相同，較無變化。

## Compare with other method: [FastPhotoStyle](https://arxiv.org/pdf/1802.06474.pdf)
- 架構
![](https://i.imgur.com/ou5fDji.jpg)

- 介紹  
FastPhotoStyle由UC Merced及NVIDIA共同提出，主要保留了content image的景色並融合style image的風格，最後生成出風格轉移的圖片。轉換方法是將content與style兩張inputs透過PhotoWCT(Whitening and Coloring Transform)來進行轉換，再利用smoothing將產生後的圖片平滑化，使圖像更加逼真。

![](https://i.imgur.com/WzjzGTj.jpg)

上圖為WCT與PhotoWCT的架構圖，其中他們的差別在於，WCT經過Max pooling layer → Upsampling layer後較無法恢復原始圖像的結構細節(如上左圖)，因此他們利用Unpooling layer取代Upsampling layer，來保留原圖的空間特色與資訊。

## Inference FastPhotoStyle
我們利用已給定的model(`photo_wct.pth`)來跑生成，而content及style image的部分使用與MUNIT相同的各10張圖片。

以下為FastPhotoStyle的生成結果：

- Content-風景照 / Style-浮世繪

![](https://i.imgur.com/RefN7le.jpg)

- Content-浮世繪 / Style-風景照

![](https://i.imgur.com/GuCWnNG.jpg)

### Analysis
- 在content為風景照、style為浮世繪的FastPhotoStyle生成圖中，由於會保留content的輪廓，再將圖片轉為style的色系，所以成果的構圖較MUNIT佳。但也由於FastPhotoStyle很完整地保留了content原本的結構，使得它生成的圖片無法擁有像浮世繪一樣的畫風與線條，只能在色調上有所轉換。而風景照的內容物不同，也會使得output的呈現有所不同：若input的風景照是由山和藍天所組成，最後的生成結果會看起來像是復古的油墨畫；但若是原圖包含了現實中的建築物或動物，結果則較無法呈現出繪畫的風格，仍然維持真實照片的樣子。
- 同樣地，content為浮世繪、style為風景圖的output，也是由於FastPhotoStyle保留了content原本的結構，使得生成結果仍維持浮世繪的風格，無法轉換成真實照片的樣子，只有在顏色上有所改變，且圖片變得較為朦朧。此外，我們猜測可能由於style採用的風景圖不像大部分浮世繪的色調大致是相近的，而是每張圖片都會因應取景的不同而有不同的鮮明色彩，所以content的圖片與選擇搭配的style圖片如果不屬於同類型的元素，轉換之後的結果即可能會更不如預期。因此若要改善生成結果，可能要試著挑選適當的組合去搭配與轉換。

## Overall Comparison
就ukiyoe2photo的轉換而言：
- 以生成的結果來說，MUNIT無論是浮世繪轉風景照或風景照轉浮世繪，成果的輪廓皆不如FastPhotoStyle。手繪圖往往是將實景複雜的輪廓簡化為簡單的線條，並加入藝術家調配的色系。透過FastPhotoStyle將風景照轉成浮世繪，能保留住風景的線條結構並加入浮世繪的色調，較容易呈現出手繪圖的感覺，即使沒有成功轉換成浮世繪的畫風，整體看起來仍比MUNIT成功；然而浮世繪的線條組成即使透過FastPhotoStyle的方法仍無法轉為實景該有的複雜結構，生成的圖片依舊是浮世繪的畫風，因此導致成果不夠理想。
- 就整體的材質來說，MUNIT的成果較好。MUNIT浮世繪轉風景照後所呈現的藍天、綠地和黃土相當類似於實際照片的質地；照片轉浮世繪的色調也確實如同真正的浮世繪。而FastPhotoStyle產生的結果，則比較像是進行色調上的轉換。
- 實際而言，以風景照轉浮世繪來說，FastPhotoStyle呈現的效果雖然較MUNIT成功，但其生成結果並未真正地將真實的風景照轉換成浮世繪的風格，而只是將原圖轉換成跟浮世繪用色相似的圖像，若只單看output而言，也較難猜測出轉換後的風格為浮世繪。

### Compare with other datasets
由於ukiyoe2photo的dataset可能並不適合用MUNIT的方式生成，因此我們試著用另外兩個的datasets─housecat2dog與edges2handbags來進行比較。我們自己上網找了的圖片當作inputs，分別進行MUNIT與FastPhotoStyle的生成，來加以分析MUNIT及FastPhotoStyle的轉換結果。（MUNIT所使用的model為pre-trained model─`housecat2dog.pt`以及`edges2handbags.pt`）

結果如下：
- MUNIT (Dog to Cat)
![](https://imgur.com/0DZDF4L.jpg)
- FastPhotoStyle (Content-Dog / Style-Cat)
![](https://imgur.com/XBObCIO.jpg)
-----------------------------------------------
- MUNIT (Cat to Dog)
![](https://imgur.com/2Aj1Q12.jpg)
- FastPhotoStyle (Content-Cat / Style-Dog)
![](https://imgur.com/qI3Hzhq.jpg)
-----------------------------------------------  
- MUNIT (Edge to Handbag)
![](https://imgur.com/VvLvODl.jpg)
- FastPhotoStyle (Content-Edge / Style-Handbag)
![](https://imgur.com/uf2iwdX.jpg)
-----------------------------------------------  
- MUNIT (Handbag to Edge)
![](https://imgur.com/808Xy28.jpg)
- FastPhotoStyle (Content-Handbag / Style-Edge)
![](https://imgur.com/Drk5cyn.jpg)

由MUNIT、FastPhotoStyle兩種方法的生成結果可以看出，housecat2dog與edges2handbags的轉換，MUNIT的效果皆較佳。以上的input透過MUNIT皆確實成功轉換型態，但透過FastPhotoStyle的結果卻只是content在色彩上有了改變。

再綜合ukiyoe2photo的轉換成果，可以得出結論：FastPhotoStyle適合用在單純只有色調轉換的圖片生成；而MUNIT則是適合用在形體、結構差不多，卻要大幅轉換其texture的圖片生成。
