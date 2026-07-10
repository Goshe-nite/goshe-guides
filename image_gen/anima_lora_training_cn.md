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

## 其他
