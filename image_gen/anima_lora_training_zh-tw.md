# Anima LoRA訓練簡介
以下一律默認Windows 11系統與Nvidia顯卡

## 下載模型
- 模型: https://huggingface.co/circlestone-labs/Anima/blob/main/split_files/diffusion_models/anima-base-v1.0.safetensors
- Text encoder: https://huggingface.co/circlestone-labs/Anima/blob/main/split_files/text_encoders/qwen_3_06b_base.safetensors
- vae: https://huggingface.co/circlestone-labs/Anima/blob/main/split_files/vae/qwen_image_vae.safetensors

## 安裝訓練器
直接全部複製貼上到cmd
```bat
git clone https://github.com/gazingstars123/Anima-Standalone-Trainer
cd Anima-Standalone-Trainer
.\setup_env.bat
```
## 準備圖片
盡量準備"乾淨"的圖片, 如無文字, 無打碼。最好能確保訓練目標(通常是角色/畫風/概念)的一致性, 並移除任何偏離目標的圖片。大小和縱橫比(aspect ratio)可以用<https://cropper.launchigniter.com/en>進行裁剪調節，或其他工具也沒問題，但是建議最長邊1024px以內, 縱橫比越少越好(一組圖片內1-3組縱橫比)。圖片數量方面, 網絡流傳角色30/畫風50/概念100, 但是實際上如果圖片品質好的話大概不需要那麼多, 不過還是建議至少30張。有需要亦可修圖或用AI來修復有問題的圖片。

## 自動打標籤
回到根目錄, 然後:
```bat
git clone https://github.com/Cyrus-Hei/timmtagger-lite.git
cd timmtagger-lite
python3.11 -m venv venv
.\venv\Scripts\activate
pip install -r requirements.txt
```
跑自動打標籤(會給每張圖生成一個含tag的txt文檔, 命名為<原圖無後綴名稱>.txt):
```bat
python batch_inference.py <圖片目錄>
```

## 訓練設定
進入`Anima-Standalone-Trainer\training-ui`shua雙擊打開`start_training_ui_anima.bat`, 如無錯誤會自動打開<http://localhost:3000/>
> [!IMPORTANT]
> 沒有提及的參數請保持原封不動
1. 左下角Global Settings設定模型路徑
2. 左上角+ New新增訓練
3. 設定主要參數 - Training
```bash
Learning Rate = 0.00007-0.0002 # 使用其他Optimizer會需要更改
Optimizer = AdamW8bit
LR Scheduler = Cosine

Duration Unit = Epochs # 按個人喜好, 1 epoch即完成一輪使用所有圖片的訓練(含重複)
Max Epochs = 60 # 視情況增加減少, 個人認為60足夠了
Max Steps = (圖片數*repeat/batch size)*max epochs # duration unit選了steps的話需要填步數, 不想算的話填2000
Save Every N Epochs = 3 # 建議至少保存10個LoRA, 比較容易找到成功的
Output Name = 喜歡甚麼填甚麼

最底
Cache Text Encoder Outputs to Disk = 不勾選 # 不太清楚該不該勾, 需要更多測試
```
4. Dataset
```bash
Resolution = 1024 # 用更大會需要更多時間, 不過有時間也不是不行, 不過不擔保品質會提升
Batch Size = 4 # 越大使用越多vram, 改動會需要調整learning rate與max epochs/steps
Gradient Accumulation = 1 # 如果不夠vram的話調整為2, batch size調整為2 (實際batch size為batch size * gradient accumulation)

Dataset Folders
Image directory = 圖片目錄
Num repeats = 2 # 重複 - 建議大約設定為100/圖片數
Caption Prefix = 如果之前沒有添加啟動標籤可以這裡填, 如`@goshenite, `
shuffle caption = 打勾
```
5. Network
```bash
Network Dim(rank) = 16
Network Alpha = 16
Train UNet only = 勾選
Auto-resume from last saved state = 勾選
```
6. 跳過multi-gpu, 打開prompts
```bash
Sample every N epochs = 3 # 可自行調整, 但太多取樣會影響拖慢訓練進度
Keep Model Loaded = Vram足夠可以勾選
Negative prompt = worst quality, low quality, score_1, score_2, score_3, early, old, blurry, jpeg artifacts, bokeh, sepia, monochrome, flat color, sketch, lowres, decensored, censored, mosaic censoring, bar censor, bad anatomy, bad proportions, deformed, extra fingers, polydactyl, malformed hands, bad hands, bad feet
Steps = 24; Scale = 3; Seed = 11111 # 可以自行調整, 但建議使用固定seed(添加positive prompt後點apply to all)
點擊+Add Prompt添加positive prompt, 輸入測試取樣用提示詞
```
7. 點右上角train開始訓練 (使用以上設定大概使用9.5 GB vram, 訓練時間視圖片數而定, 我測試中3090 - 50張圖大概是兩小時)

## 其他
