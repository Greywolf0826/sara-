from flask import Flask, render_template, request, jsonify
import openai
import json
import os

# OpenAI API anahtarını buraya ekleyin
OPENAI_API_KEY = "SENİN_OPENAI_API_ANAHTARIN"
openai.api_key = OPENAI_API_KEY

app = Flask(__name__)

def sohbet_ai(mesaj, sohbet_gecmisi):
    """
    Kullanıcı mesajına göre OpenAI GPT modelini çağıran fonksiyon.
    """
    sohbet_gecmisi.append({"role": "user", "content": mesaj})
    yanit = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",  # Burada model versiyonunu değiştirebilirsin
        messages=sohbet_gecmisi
    )
    yanit_mesaji = yanit["choices"][0]["message"]["content"]
    sohbet_gecmisi.append({"role": "assistant", "content": yanit_mesaji})
    return yanit_mesaji

# Sohbet geçmişini saklamak için boş bir liste
sohbet_gecmisi = []

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/sohbet', methods=['POST'])
def sohbet():
    kullanici_verisi = request.get_json()
    mesaj = kullanici_verisi.get("mesaj")
    yanit = sohbet_ai(mesaj, sohbet_gecmisi)
    return jsonify({"yanit": yanit})

if __name__ == '__main__':
    app.run(debug=True)

# HTML ve JavaScript Kısmı
# templates/index.html dosyası oluşturulmalı ve aşağıdaki kod eklenmeli:

# <!DOCTYPE html>
# <html lang="tr">
# <head>
#     <meta charset="UTF-8">
#     <meta name="viewport" content="width=device-width, initial-scale=1.0">
#     <title>AI Sohbet Botu</title>
#     <script>
#         async function mesajGonder() {
#             let kullaniciMesaji = document.getElementById("mesaj").value;
#             let sohbetKutusu = document.getElementById("sohbet");
#             sohbetKutusu.innerHTML += "<p><b>Sen:</b> " + kullaniciMesaji + "</p>";
#             document.getElementById("mesaj").value = "";

#             let yanit = await fetch("/sohbet", {
#                 method: "POST",
#                 headers: {"Content-Type": "application/json"},
#                 body: JSON.stringify({mesaj: kullaniciMesaji})
#             });

#             let yanitJson = await yanit.json();
#             sohbetKutusu.innerHTML += "<p><b>AI:</b> " + yanitJson.yanit + "</p>";
#         }
#     </script>
# </head>
# <body>
#     <h2>AI Sohbet Botu</h2>
#     <div id="sohbet" style="border: 1px solid black; width: 50%; height: 300px; overflow-y: scroll; padding: 10px;"></div>
#     <input type="text" id="mesaj" placeholder="Mesajınızı yazın...">
#     <button onclick="mesajGonder()">Gönder</button>
# </body>
# </html>
