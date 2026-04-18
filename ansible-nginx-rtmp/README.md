# ansible-nginx-rtmp

Ansible playbook to provision an Nginx + RTMP streaming server on Alpine Linux, tuned for **LAN use with low-resource hosts** (2 vCPU / 2 GB RAM class, no GPU). Passthrough only — no transcoding.

Typical use case: OBS Studio pushes an x264 RTMP stream into the server, one or two concurrent feeds, played back over LAN with mpv/VLC (RTMP) or a browser (HLS).

## 🧰 Requirements

- Ansible >= 2.14
- Collection `community.general`: `ansible-galaxy collection install community.general`
- Alpine Linux on the target host
- SSH key configured on the target

## ⚙️ Setup

```bash
cp hosts.ini.example hosts.ini
cp group_vars/all.yml.example group_vars/all.yml
# edit hosts.ini and group_vars/all.yml
```

## 🔧 Configuration

Key variables in `group_vars/all.yml`:

| Variable | Default | Purpose |
|---|---|---|
| `rtmp_app_name` | `stream` | RTMP application name (ingest path) |
| `rtmp_port` | `1935` | RTMP listen port |
| `http_port` | `80` | HTTP listen port (HLS, stats) |
| `rtmp_chunk_size` | `8192` | RTMP chunk size, tuned for LAN |
| `rtmp_allow_cidr` | `""` | If set (e.g. `192.168.0.0/24`), restricts publish/play to that subnet |
| `hls_enabled` | `true` | Toggle HLS output and `/hls` endpoint |
| `hls_fragment` | `2` | HLS segment length, seconds |
| `hls_playlist_length` | `10` | HLS playlist window, seconds |
| `dash_enabled` | `false` | Toggle DASH output and `/dash` endpoint |
| `stat_enabled` | `true` | Toggle `/stat` RTMP stats page |

## ▶️ Run

```bash
ansible-playbook -i hosts.ini playbook.yml
```

Dry-run / syntax check:

```bash
ansible-playbook -i hosts.ini playbook.yml --check
ansible-playbook -i hosts.ini playbook.yml --syntax-check
```

## 📥 Publish (OBS)

In OBS → Settings → Stream:

- Service: **Custom**
- Server: `rtmp://<host>:1935/<rtmp_app_name>`
- Stream key: your key (e.g. `live`)

In OBS → Settings → Output (advanced), for HLS to start cleanly:

- Encoder: **x264**
- **Keyframe interval: 2 s** (must match `hls_fragment`)

## 📤 Playback

Lowest-latency (≈ 0.5–1 s on LAN), recommended when possible:

```bash
mpv rtmp://<host>:1935/<rtmp_app_name>/<key>
vlc rtmp://<host>:1935/<rtmp_app_name>/<key> --network-caching=200
```

HLS, for browsers or when RTMP isn't available:

```
http://<host>/hls/<key>.m3u8
```

## 🌐 Endpoints

| URL | When |
|---|---|
| `rtmp://host:1935/<app>/<key>` | always — ingest & direct playback |
| `http://host/hls/<key>.m3u8` | `hls_enabled: true` |
| `http://host/dash/<key>.mpd` | `dash_enabled: true` |
| `http://host/stat` | `stat_enabled: true` |
| `http://host/health` | always — returns `ok` |

## ⏱️ Latency

Reference values, end-to-end from OBS to player on LAN:

| Protocol | Typical | Supported here |
|---|---|---|
| RTMP direct | 0.5–1 s | ✅ |
| HLS (this config) | 4–6 s | ✅ |
| LL-HLS | 2–3 s | ❌ not in stock nginx-rtmp |
| HTTP-FLV | 1–2 s | ❌ requires a different nginx build |
| WebRTC | < 1 s | ❌ |

HLS has a structural floor around 4–6 s because the player needs several fragments before playback starts. For lower latency on LAN, use RTMP playback directly.

## 🎯 When this playbook is the right tool

- You need a lightweight RTMP relay on LAN with minimal overhead.
- You don't need transcoding, ABR, or a web UI.
- You're happy with ~5 s HLS latency or you'll play over RTMP directly.

If you need sub-3-s HLS, WebRTC, or a management UI, consider [MediaMTX](https://github.com/bluenviron/mediamtx) or [SRS](https://github.com/ossrs/srs) instead — they address a different point in the trade-off space.
