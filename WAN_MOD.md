# Мод: передача файлов через интернет

Личный форк [LocalSend](https://github.com/localsend/localsend) (Apache-2.0).
Цель: отправлять файлы между устройствами в разных сетях, например из дома папе в другой город, без ручного ввода IP и проброса портов.

## Что уже есть в upstream (на 2026-10-30, коммит `92e133aa`)

- **WebRTC-передача уже реализована, но выключена**:
  `const webRTCEnabled = false;` в `app/lib/provider/network/webrtc/signaling_provider.dart`.
- Ядро на Rust: `packages/core/src/webrtc/` (feature `webrtc`, `webrtc-signaling`).
- Сигнальный сервер: `server/` (Axum, WebSocket `/v1/ws`), публичный — `wss://public.localsend.org/v1/ws`.
- STUN по умолчанию: `stun:stun.localsend.org:3478`.
- **Ограничение:** сервер группирует устройства по внешнему IP (`server/src/util/ip.rs`: IPv4 — один адрес, IPv6 — /64).
  Устройства видят друг друга, только если у них один внешний IP. Устройства в разных городах друг друга не найдут.

## Что нужно сделать (черновик, уточнить на этапе дизайна)

1. Включить WebRTC и проверить, что передача работает внутри одной IP-группы.
2. Добавить группировку по коду комнаты или паре: оба устройства вводят один код, сервер сводит их, даже если внешние IP разные.
3. Свой сигнальный сервер (Docker из `server/Dockerfile`), адрес задаётся в настройках.
4. TURN-ретранслятор на случай, когда P2P не пробивается через NAT (например, coturn).
5. UI: ввод и показ кода, статус соединения.

## Ограничения

- Лицензия Apache-2.0: сохранять `LICENSE`, помечать изменённые файлы.
  Название и логотип LocalSend при распространении не использовать — нужно своё имя.
- Upstream не принимает крупные PR, сгенерированные ИИ (см. `AGENTS.md`).
  Этот мод — для себя. В upstream — только обсуждение или мелкие исправления.

## Синхронизация с upstream

```bash
git fetch upstream
git checkout main && git merge --ff-only upstream/main && git push origin main
git checkout feat/wan-transfer && git merge main
```

Push в `upstream` отключён (`no_push`). GitHub Actions в этом репозитории выключены.

## Сборка (Linux)

Требуется: `fvm` + Flutter из `.fvmrc`, Rust из `rust-toolchain.toml`, `clang`, `cmake`, `ninja-build`, `libgtk-3-dev`.
Команды — в `AGENTS.md`. Всегда `fvm flutter` / `fvm dart`, не голые `flutter` / `dart`.
