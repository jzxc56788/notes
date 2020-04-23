# word2vec
-	推論手法是以推測為目標，獲得字詞分散式表現這個副產物。
- 	word2vec 屬於推論手法，由簡單的雙層類神經網路構成。
-  	word2vec 包括 skip-gram 模型與 CBOW 模型。
-   	相反地，skip-gram 模型是用一個字詞（目標對象）推測多個字詞（上下文）。
-    word2vec 可以重新學習權重，能有效地更新或新增字詞的分散式表示。

---
- 	Embedding 層會儲存字詞的分散式表示，在正向傳播中，擷取該字詞 ID 的向量。
-  在 word2vec 中，語彙量的增加程度與運算量成正比，因此最好採取可以執行近似運算的快速手法。
-  Negative Sampling 是取樣幾個負例的手法，利用這種手法，可以把多值分類當坐二值分類處理。
-  利用 word2vec 獲得的字詞分散式表示嵌入了詞意：而當作類似上下文使用的字詞，在詞向量的空間上，位於相近的位置。
-  word2vec 的字詞分散式表示具有利用向量的加、減法，解答類推問題的性質。
-  word2vec 的遷移學習是格外重要的關鍵，字詞的分散式表示可以運用在各式各樣的自然語言任務上。

---

## 計數手法的問題
利用周圍的詞頻來表現字詞，就是建立 **共生矩陣**。針對該矩陣套用 **SVD** 獲得稠密矩陣（字詞的分散式表示）。<br>
SVD 是針對 $$n \times n$$ 的矩陣，花費 $$O(n^3)$$ 的計算成本。<br>
假設語彙量超過 100萬，在計數手法中，會建立 100萬 $$\times$$ 100萬的巨大矩陣。過於龐大。

計數手法：一次 **統一** 處理學習資料。<br>
推論手法：使用 **部分** 學習資料，逐次學習。

推論手法可以分批學習且可以平行運算，整理學習也會比較高速化。

## 推論手法的概要
當給予周圍的字詞（上下文）時，推測『？』會出現哪些字詞與其機率。以類神經網路當作模型進行學習，最後可以獲得字詞的分散式表示，當作學習結果，這就是推論手法的整體概念。

>	<u>you</u> { ? } <u>goodbye</u> and I say hello.

推論手法與計數手法都是以 **分布假說** 為基礎。分布假說：『字詞的意思是由周圍的字詞形成的』

## 類神經網路的字詞處理方式
類神經網路無法直接處理「you」、「say」等字詞，必須轉換成「固定長度的向量」。<br>
ex：one-hot 編碼

如果語料庫有七個字詞。

字詞	|字詞ID	|one-hot 編碼
---		|---	|---
you		|0		|(1,0,0,0,0,0,0)
goodbye|2		|(0,0,1,0,0,0,0)
輸入層的神經元會有七個（語料庫有幾個字詞就有幾個 分別對應），中間層為 3 的話就需要一個$$Ｗ(7,3)$$矩陣（**很沒效率**）

## 簡化型 word2vec
$$I_1$$：上文 <br>
$$I_2$$：下文<br>
$$W_{in}$$：第一層的權重，學習後會是字詞的分散式表示

$$h_1 = I_1 \times W_{in}\\h_2 = I_2 \times W_{in}\\H=\frac{h_1+h_2}{2}\\O=H \times W_{out}$$

$$H$$：中間層 <br>
$$O$$：輸出層<br>
機率：$$Softmax(O)$$

減少$$H$$的數量才是關鍵，中間層必須『精簡』存放預測字詞的必要資料稠密向量。

-	加入新字詞：計數手法，從0開始計算；
				推論手法，目前的$$W$$在學習

##	改善 word2vec
### Embedding 層
因為 one-hot 佔用大量的記憶體，有會耗費大量計算資源。

取出「字詞 ID 該列（向量）」用的分層，作 Embedding 層。

如果 id 陣列中，有相同 id 時用加法。

### Negative Sampling（負取樣）
-	取代 Softmax 可以抑制運算量
- 	從多值分類變成二值分類，用「Yes/No」回答問題<br>
	「當上下文是『you』及『goodbye』時，成為目標對象的字詞是『say』嗎？」，這樣輸出層只要準備一個神經元就好。
-  	解決二分類問題，就要用到 Sigmoid。Sigmoid 與 Cross Entropy Error，整合成 Sigmoid with Loss。
-   	除了看正例也要看反例，只取樣少量反例。<br>
	-	取樣機率：$$P'(w_i) = \frac{P(w_i)^{0.75}}{\sum^n_j{P(w_i)^{0.75}}}$$
		-	$$P(w_i)$$代表第$$i$$個字詞的機率
		- 	乘上某次方後重新標準化
		- 	0.75 並沒有任何理由，可以更改。