import os, re, time, secrets, threading, unicodedata
from urllib.parse import quote
from flask import Flask, request, jsonify, Response, render_template, abort, stream_with_context
import yt_dlp
import requests as rq

app = Flask(__name__)

UA = ("Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 "
      "(KHTML, like Gecko) Chrome/126.0 Safari/537.36")
DOMINIOS = ("pornhub.com", "pornhub.org")
TTL = 600
MAX_SIMULTANEAS = 4

_cache = {}
_lock = threading.Lock()
_slots = threading.Semaphore(MAX_SIMULTANEAS)
_ip_ultima = {}


def permitido(url: str) -> bool:
    m = re.match(r"^https?://([^/]+)", url or "")
    if not m:
        return False
    host = m.group(1).lower().removeprefix("www.").removeprefix("m.")
    return any(host == d or host.endswith("." + d) for d in DOMINIOS)


def extraer(url: str):
    opts = {"quiet": True, "no_warnings": True, "noplaylist": True,
            "skip_download": True, "socket_timeout": 20}
    if os.path.exists("cookies.txt"):
        opts["cookiefile"] = "cookies.txt"
    with yt_dlp.YoutubeDL(opts) as ydl:
        info = ydl.extract_info(url, download=False)
    if info.get("_type") == "playlist":
        info = info["entries"][0]
    return info


def limpio(t: str, n: int = 80) -> str:
    t = unicodedata.normalize("NFKD", t or "video")
    t = "".join(c for c in t if not unicodedata.combining(c))
    return re.sub(r"[^\w\s.-]", "", t).strip().replace(" ", "_")[:n] or "video"


@app.post("/api/info")
def info():
    ip = request.headers.get("X-Forwarded-For", request.remote_addr or "?").split(",")[0].strip()
    ahora = time.time()
    if ahora - _ip_ultima.get(ip, 0) < 4:
        return jsonify(error="Espera unos segundos entre enlaces."), 429
    _ip_ultima[ip] = ahora

    url = (request.get_json(silent=True) or {}).get("url", "").strip()
    if not permitido(url):
        return jsonify(error="Pega un enlace de Pornhub."), 400

    try:
        d = extraer(url)
    except Exception as e:
        return jsonify(error=f"No se pudo leer el vídeo: {str(e)[:160]}"), 502

    vistos, fmt = set(), []
    for f in d.get("formats", []):
        if f.get("vcodec") in (None, "none") or f.get("ext") != "mp4":
            continue
        h = f.get("height") or 0
        if h in vistos or not f.get("url"):
            continue
        vistos.add(h)
        peso = f.get("filesize") or f.get("filesize_approx")
        fmt.append({"fid": f["format_id"], "h": h, "url": f["url"],
                    "mb": round(peso / 1048576) if peso else None})
    fmt.sort(key=lambda x: x["h"], reverse=True)

    if not fmt:
        return jsonify(error="Este vídeo no ofrece mp4 descargable."), 422

    sid = secrets.token_urlsafe(12)
    with _lock:
        for k in [k for k, v in _cache.items() if v["t"] < ahora - TTL]:
            _cache.pop(k, None)
        _cache[sid] = {"t": ahora, "titulo": d.get("title") or "video",
                       "thumb": d.get("thumbnail"), "dur": d.get("duration") or 0,
                       "fmt": fmt, "url": url}

    return jsonify(id=sid, titulo=d.get("title") or "video", thumb=d.get("thumbnail"),
                   dur=d.get("duration") or 0,
                   calidades=[{"fid": x["fid"], "h": x["h"], "mb": x["mb"]} for x in fmt])


@app.get("/descargar/<sid>/<fid>")
def descargar(sid, fid):
    d = _cache.get(sid)
    if not d or d["t"] < time.time() - TTL:
        abort(410, "Enlace caducado, vuelve a pegar la URL.")
    f = next((x for x in d["fmt"] if x["fid"] == fid), None)
    if not f:
        abort(404)

    if not _slots.acquire(blocking=False):
        abort(429, "Servidor ocupado, reintenta en unos segundos.")

    try:
        up = rq.get(f["url"], headers={"User-Agent": UA, "Referer": "https://www.pornhub.com/"},
                    stream=True, timeout=30)
        up.raise_for_status()
    except Exception:
        _slots.release()
        abort(502, "El CDN no respondió.")

    nombre = f"{limpio(d['titulo'])}_{f['h']}p.mp4"

    def gen():
        try:
            for trozo in up.iter_content(262144):
                yield trozo
        finally:
            up.close()
            _slots.release()

    return Response(stream_with_context(gen()), mimetype="video/mp4", headers={
        "Content-Disposition": f"attachment; filename=\"{nombre}\"; filename*=UTF-8''{quote(nombre)}",
        "Content-Length": up.headers.get("Content-Length", ""),
        "Cache-Control": "no-store",
        "X-Accel-Buffering": "no",
    })


@app.get("/")
def home():
    return render_template("index.html")


if __name__ == "__main__":
    app.run(port=5000, threaded=True)
