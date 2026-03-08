import ui
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

# AES 設定：金鑰必須是 16, 24 或 32 位元組
BLOCK_SIZE = 16

class CryptoGUI(ui.View):
    def __init__(self):
        self.setup_ui()

    def setup_ui(self):
        self.name = 'AES 安全加密工具'
        self.background_color = '#f7f7f7'
        
        # 1. 原始文字輸入
        self.input_text = ui.TextView(frame=(20, 40, 300, 100))
        self.input_text.border_width = 1
        self.input_text.placeholder = '在此輸入要加密或解密的文字...'
        self.add_subview(self.input_text)
        
        # 2. 金鑰輸入 (Key) - 必須是 16 個字元
        self.key_field = ui.TextField(frame=(20, 150, 300, 40))
        self.key_field.placeholder = '輸入 16 位元金鑰 (例如: 1234567890123456)'
        self.key_field.text = 'mysecretkey12345'
        self.add_subview(self.key_field)
        
        # 3. 加密按鈕
        self.enc_btn = ui.Button(frame=(20, 210, 140, 50))
        self.enc_btn.title = '執行 AES 加密'
        self.enc_btn.background_color = '#007aff'
        self.enc_btn.tint_color = 'white'
        self.enc_btn.action = self.encrypt_action
        self.add_subview(self.enc_btn)
        
        # 4. 解密按鈕
        self.dec_btn = ui.Button(frame=(180, 210, 140, 50))
        self.dec_btn.title = '執行 AES 解密'
        self.dec_btn.background_color = '#ff9500'
        self.dec_btn.tint_color = 'white'
        self.dec_btn.action = self.decrypt_action
        self.add_subview(self.dec_btn)
        
        # 5. 結果顯示
        self.result_text = ui.TextView(frame=(20, 280, 300, 100))
        self.result_text.editable = False
        self.result_text.background_color = '#e5e5ea'
        self.add_subview(self.result_text)

    def get_key(self):
        key = self.key_field.text.encode('utf-8')
        # 確保金鑰長度為 16 (AES-128)
        return key[:16].ljust(16, b'\0')

    def encrypt_action(self, sender):
        try:
            data = self.input_text.text.encode('utf-8')
            key = self.get_key()
            cipher = AES.new(key, AES.MODE_ECB) # 使用簡單的 ECB 模式
            ct_bytes = cipher.encrypt(pad(data, BLOCK_SIZE))
            self.result_text.text = base64.b64encode(ct_bytes).decode('utf-8')
        except Exception as e:
            self.result_text.text = f"加密出錯: {e}"

    def decrypt_action(self, sender):
        try:
            data = base64.b64decode(self.input_text.text)
            key = self.get_key()
            cipher = AES.new(key, AES.MODE_ECB)
            pt = unpad(cipher.decrypt(data), BLOCK_SIZE)
            self.result_text.text = pt.decode('utf-8')
        except Exception as e:
            self.result_text.text = f"解密失敗 (檢查金鑰或內容): {e}"

# 啟動介面
v = CryptoGUI()
v.present('sheet')
