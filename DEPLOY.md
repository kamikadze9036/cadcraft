# Nasazení SolidCraft (webová verze) na domácí server

Webová verze je čistě statická (WebAssembly), nepotřebuje žádný backend.
Zdroj: `apps/solidcraft-web`, postup podle `docs/building.md`.

## 1. Předpoklady na serveru

- Git, C toolchain (`build-essential`)
- Rust přes rustup + wasm target a trunk:

```bash
rustup target add wasm32-unknown-unknown
cargo install trunk --locked
```

- Hodně volného místa na disku (build ve `target/` má desítky GB).
  Pro menší build: `CARGO_PROFILE_DEV_DEBUG=0 CARGO_INCREMENTAL=0`.

## 2. Build

```bash
git clone https://github.com/kamikadze9036/cadcraft.git
cd cadcraft/apps/solidcraft-web
trunk build --release --public-url ./
```

Výstup je ve složce `dist/` (relativní cesty, funguje v libovolné podcestě).

## 3. Servírování

Service worker (`sw.js`) a WebGPU fungují jen přes **HTTPS** nebo na `localhost`.
Nejjednodušší je Caddy (certifikát si zařídí sám, pro doménu v LAN použij vlastní CA
nebo `tls internal`).

`Caddyfile`:

```
cad.example.lan {
    root * /srv/solidcraft
    file_server
    encode zstd gzip
    tls internal
}
```

```bash
sudo mkdir -p /srv/solidcraft
sudo cp -r dist/* /srv/solidcraft/
sudo systemctl reload caddy
```

Alternativa nginx: `root /srv/solidcraft;`, `try_files $uri $uri/ =404;`,
`types { application/wasm wasm; }` (MIME pro `.wasm` musí být správný) a HTTPS certifikát.

## 4. Použití

- Otevři `https://cad.example.lan/`
- `?sample` otevře ukázkový díl
- `?webgl` použije WebGL2, když prohlížeč nemá WebGPU
- Jazyk (včetně češtiny) se přepíná v nastavení aplikace

## 5. Aktualizace

```bash
cd cadcraft && git pull
cd apps/solidcraft-web && trunk build --release --public-url ./
sudo cp -r dist/* /srv/solidcraft/
```

Service worker si drží starou verzi v cache, po aktualizaci udělej tvrdé obnovení
stránky (Ctrl+Shift+R).

## Známá omezení

- Prohlížečová aplikace zatím nečte FreeCAD/Rhino soubory (viz `docs/step-import.md`).
- Při panice jádra webová verze spadne (viz `docs/crash-reports.md`), práci si ukládej.
