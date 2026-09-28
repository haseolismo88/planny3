# Planny — pakej pembetulan Vercel

## Apa yang dibaiki

Ralat asal:

> The pattern "api/dialogue.js" defined in `functions` doesn't match any Serverless Functions inside the `api` directory.

Fail API memang ada dalam folder tempatan yang disemak. Ini bermaksud ralat bukan bukti fail tempatan itu tiada; kandungan yang dihantar, Root Directory atau pengesanan framework dalam deployment mungkin berbeza. Tetapan projek Vercel sebenar belum diperiksa.

Pakej ini menggantikan pengesanan automatik dan aturan `functions` dengan pembinaan eksplisit: `index.html` menggunakan `@vercel/static`, manakala `api/dialogue.js` menggunakan `@vercel/node`. URL `/api/dialogue` dipetakan terus kepada function tersebut. Tempoh maksimum 120 saat ditetapkan dalam eksport `config` function. Dialog dan paparan asal dikekalkan.

`builds` ialah konfigurasi legasi Vercel yang masih didokumenkan; ia digunakan di sini untuk menentukan dua fail binaan secara jelas. Jangan tambah semula bahagian `functions` ke konfigurasi ini. Amaran bahawa Build Settings dashboard tidak digunakan kerana `builds` ditetapkan adalah dijangka, bukan ralat build.

## Deploy langkah demi langkah

1. Extract ZIP ini ke folder baharu.
2. Upload **semua fail DAN folder** yang diekstrak ke repository GitHub yang digunakan oleh Vercel. Jangan upload fail ZIP sahaja, dan jangan pilih hanya enam fail di aras utama.
3. Pastikan struktur dalam repository ialah:

   ```text
   index.html
   vercel.json
   package.json
   api/
     dialogue.js
   scripts/
     verify-deploy.mjs
   ```

4. Di Vercel → Project Settings → Build and Deployment:
   - Framework Preset: **Other**.
   - Root Directory: folder yang mengandungi `vercel.json`, `package.json`, `index.html` dan folder `api` bersama-sama. Jika semuanya di akar repository, gunakan akar repository. **Jangan pilih `api`, `scripts`, `public` atau `dist`.**
   - Matikan override Build Command, Install Command dan Output Directory yang lama. Konfigurasi dalam pakej ini menentukan hasil deployment.
   - Node.js: **22.x**.
5. Environment Variables: tetapkan `OPENAI_API_KEY` dengan key anda. Pilihan: `OPENAI_MODEL=gpt-4.1-mini`. Jangan masukkan key ke HTML atau commit key dalam repository.
6. Deploy commit baharu. Untuk cubaan pertama pakej pembetulan ini, jangan guna semula Build Cache jika pilihan itu tersedia.

Jika anda memilih Redeploy pada deployment lama, Vercel mungkin membina commit lama lagi. Pastikan deployment menggunakan commit yang mengandungi `vercel.json` baharu ini.

## Cara sahkan selepas deploy

- Buka URL laman: Planny sepatutnya muncul.
- Buka `https://DOMAIN-ANDA/api/dialogue`: respons `405` dengan mesej `Gunakan POST.` ialah hasil yang **betul** apabila dibuka di browser. Ia mengesahkan function sudah wujud; ujian GET ini tidak memanggil OpenAI atau menggunakan kredit API.
- Kemudian isi produk dan tekan **Buatkan ayat video saya**. Jika key belum dipasang, mesej `OPENAI_API_KEY` akan muncul. Jika fungsi tidak wujud, semak Root Directory dan pastikan `api/dialogue.js` benar-benar ada dalam commit yang dideploy.

Jika ralat lama yang menyebut `functions` masih muncul, deployment masih membaca konfigurasi lama atau konfigurasi dari folder lain: fail `vercel.json` dalam ZIP ini tidak mempunyai bahagian `functions`.

## Fungsi Planny

- OpenAI menghasilkan dialog berdasarkan input produk dan prompt gambar/video setiap part.
- Setiap part disemak tepat 18 perkataan, dengan pembetulan automatik jika perlu.
- 120 idea setiap set; sehingga 100 set / 12,000 ID variasi bagi setiap nama produk.
- Semakan ayat sama berdasarkan sejarah browser ini sahaja. Keunikan makna atau keunikan merentas pengguna/peranti tidak dijamin.
- Kemajuan separa disimpan; tekan semula dengan input dan pilihan yang sama untuk sambung. Biarkan tab terbuka semasa janaan.
- Paparan menyimpan set lengkap terkini. Muat turun set sebelum menjana set baharu jika mahu menyimpan semuanya. Memadam data browser memadam sejarah.

## Pengesahan pakej

Jalankan `npm run verify` untuk menyemak fail binaan, laluan API, eksport function dan sintaks skrip HTML. `npm test` menguji API dengan respons simulasi tanpa menggunakan key atau kredit sebenar.

Pakej telah disemak secara tempatan. Build pada pelayan Vercel dan panggilan OpenAI sebenar belum diuji dalam akaun anda.

Rujukan: [konfigurasi builds Vercel](https://vercel.com/docs/project-configuration/vercel-json#builds), [pelaksanaan Node builder Vercel](https://github.com/vercel/vercel/blob/main/packages/node/src/build.ts).
