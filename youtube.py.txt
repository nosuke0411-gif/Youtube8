from flask import Flask, request, jsonify

app = Flask(__name__)

def convert_youtube_url(url: str) -> str:
    base_mobile = "https://m.youtube.com/watch?v="
    base_pc = "https://www.youtube.com/watch?v="

    if url.startswith(base_mobile):
        video_id = url[len(base_mobile):]
        return f"https://youtu.be/{video_id}"
    elif url.startswith(base_pc):
        video_id = url[len(base_pc):]
        return f"https://youtu.be/{video_id}"
    else:
        raise ValueError("対応していないURL形式です")

@app.route("/")
def index():
    return """
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>YouTube URL 変換ツール</title>
<style>
    body {
        font-family: sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        margin: 0;
        background: #f7f7f7;
        transition: background 0.3s, color 0.3s;
    }
    body.dark {
        background: #1e1e1e;
        color: #f1f1f1;
    }

    .container {
        position: relative; /* ← これで左上に配置できる */
        text-align: center;
        background: white;
        padding: 40px;
        border-radius: 12px;
        box-shadow: 0 0 15px rgba(0,0,0,0.1);
        width: 90%;
        max-width: 500px;
        transition: background 0.3s, color 0.3s;
    }
    body.dark .container {
        background: #2c2c2c;
        color: #f1f1f1;
    }

    /* カード左上の極小ダークモードボタン */
    #darkBtn {
        position: absolute;
        top: 10px;
        left: 10px;
        padding: 2px 6px;
        font-size: 14px;
        border-radius: 4px;
        background: #6c757d;
        color: white;
        border: none;
        cursor: pointer;
        height: 22px;
        width: 22px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    /* 入力欄＋✖️（9:1比率） */
    .input-area {
        display: flex;
        align-items: center;
        gap: 6px;
    }

    input {
        flex: 9;
        padding: 14px;
        font-size: 18px;
        border-radius: 8px;
        border: 1px solid #ccc;
    }

    #clearInputBtn {
        flex: 1;
        padding: 6px;
        font-size: 14px;
        background: #dc3545;
        color: white;
        border: none;
        border-radius: 6px;
        cursor: pointer;
    }

    button {
        padding: 14px 20px;
        font-size: 18px;
        margin-top: 15px;
        border: none;
        border-radius: 8px;
        cursor: pointer;
        width: 100%;
    }

    #convertBtn { background: #007bff; color: white; }
    #copyBtn { background: #28a745; color: white; display: none; }

    #result {
        margin-top: 20px;
        font-size: 20px;
        font-weight: bold;
        word-break: break-all;
    }

    #history {
        margin-top: 30px;
        text-align: left;
        font-size: 16px;
        max-height: 200px;
        overflow-y: auto;
    }
    #history div {
        background: #efefef;
        padding: 10px;
        border-radius: 6px;
        margin-bottom: 8px;
        word-break: break-all;
    }
    body.dark #history div {
        background: #3a3a3a;
    }
</style>
</head>
<body>

<div class="container">

    <button id="darkBtn" onclick="toggleDark()">🌙</button>

    <h1>YouTube URL 変換ツール</h1>

    <div class="input-area">
        <input id="urlInput" type="text" placeholder="URLを入力">
        <button id="clearInputBtn" onclick="clearInput()">✖️</button>
    </div>

    <button id="convertBtn" onclick="convert()">変換する</button>
    <button id="copyBtn" onclick="copyResult()">コピーする</button>

    <p id="result"></p>

    <div id="history">
        <h3>変換履歴</h3>
    </div>
</div>

<script>
    let historyList = [];

    async function convert() {
        const url = document.getElementById("urlInput").value;

        const res = await fetch("/convert", {
            method: "POST",
            headers: {"Content-Type": "application/json"},
            body: JSON.stringify({url})
        });

        const data = await res.json();

        if (data.success) {
            document.getElementById("result").innerText = data.converted;
            document.getElementById("copyBtn").style.display = "block";

            historyList.unshift(data.converted);
            updateHistory();
        } else {
            document.getElementById("result").innerText = "エラー: " + data.error;
            document.getElementById("copyBtn").style.display = "none";
        }
    }

    function copyResult() {
        const text = document.getElementById("result").innerText;
        navigator.clipboard.writeText(text);
    }

    function clearInput() {
        document.getElementById("urlInput").value = "";
    }

    function updateHistory() {
        const historyDiv = document.getElementById("history");
        historyDiv.innerHTML = "<h3>変換履歴</h3>";

        historyList.slice(0, 20).forEach(item => {
            const div = document.createElement("div");
            div.innerText = item;
            historyDiv.appendChild(div);
        });
    }

    function toggleDark() {
        document.body.classList.toggle("dark");

        const btn = document.getElementById("darkBtn");
        if (document.body.classList.contains("dark")) {
            btn.innerText = "☀️";
        } else {
            btn.innerText = "🌙";
        }
    }
</script>

</body>
</html>
"""

@app.route("/convert", methods=["POST"])
def convert():
    data = request.json
    url = data.get("url")

    try:
        result = convert_youtube_url(url)
        return jsonify({"success": True, "converted": result})
    except ValueError as e:
        return jsonify({"success": False, "error": str(e)}), 400

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=10000)
