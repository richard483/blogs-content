Raven API Client adalah sebuah _library_ yang dibuat berdasarkan _module_ API Client Module yang terdapat di dalam [Blibli backend framework](https://github.com/bliblidotcom/blibli-backend-framework/tree/master) yang di buat menggunakan Java 8 dan Spring Boot versi 2.3, namun saya _improvisasi_ di Raven API Client dengan menggunakan Java 17 dan Spring Boot versi 3.3 dan 2.7. Improvisasi yang saya buat bisa dibilang tidaklah banyak, karena memang tujuan awal sebenarnya saya membuat API ini adalah untuk mempelajari lebih dalam mengenai _annotation_ dan Spring Boot Webflux lebih dalam. Tetapi seiring berjalannya, \*saya menyadari ada bagian dari library Api Client Blibli belum _fully reactive_ (setidaknya di versi 0.0.8).

## Test Scenario

Dalam melakukan testing, saya membuat sebuah _project_ Spring Boot sederhana yang memiliki 2 _endpoint_ yang berguna untuk melakukan testing HTTP _request_ GET dan POST. Project yang digunakan identik, hanya terdapat perbedaan di _library_ yang digunakan untuk melakukan API _call_.

Mengingat bahwa _library_ Blibli Backend Framework terakhir di _maintain_ di Java versi 8 dan menggunakan Spring Boot versi 2.3, saya perlu untuk menyesuaikan library Raven API agar mendukung versi tersebut dengan merilis versi 0.1.1-s2 yang menggunakan Spring Boot versi 2.7, sehingga dapat digunakan di Spring Boot 2.7. (Karena pada dasarnya Raven API Client ini dibangun menggunakan _library_ Spring Boot versi 3.3 dan Java 17).

Untuk _project_ yang digunakan untuk _testing_ dapat dibuka pada halaman [Github Repository berikut](https://github.com/richard483/raven-benchmark). Terdapat beberapa _branch_ disana, "blibli-client" untuk _project_ yang menggunakan API client dari Blibli, "raven-client-spring2" yang menggunakan API client saya yang sudah disesuaikan untuk Spring Boot versi 2.7, dan terdapat pula _branch_ "server" sebagai penyedia konten.

Untuk melakukan _load testing_ nya sendiri, saya menggunakan tools yang bernama k6 yang _scriptnya_ saya sertakan di bawah ini.

### K6 Script

```js
import http from "k6/http";
import { check } from "k6";
import { randomIntBetween } from "https://jslib.k6.io/k6-utils/1.2.0/index.js";

import { htmlReport } from "https://raw.githubusercontent.com/benc-uk/k6-reporter/main/dist/bundle.js";

export const options = {
  stages: [
    { duration: "10s", target: 10 },
    { duration: "10s", target: 2500 },
    { duration: "30s", target: 2500 },
    { duration: "10s", target: 0 },
  ],
};

export default function () {
  // const url = "http://localhost:8080/demo/hello";
  const randomInt = randomIntBetween(1, 10);
  const url = `http://localhost:8080/demo/${randomInt}`;

  const params = {
    headers: {
      "Content-Type": "application/json",
    },
  };

  const res = http.get(url, params);

  check(res, {
    "response code is 200": (r) => {
      return r.status === 200;
    },
    // hello
    // "body is same as expected": (r) => {
    //   return r.body === "HELLO";
    // },

    "body is same as expected": (r) => {
      return r.body === `GOT ${randomInt}`;
    },
  });
}

export function handleSummary(data) {
  return {
    "summary.html": htmlReport(data),
  };
}
```

_Script_ k6 di jalankan sebanyak 3 kali, dengan _environment_ yang diusahakan untuk se-identik mungkin, dengan selalu me-_restart service server_ yang digunakan di awal test.

## Test Result

## To wrap things up

\* : contohnya pada method berikut yang terdapat di [Api Client Blibli](https://github.com/bliblidotcom/blibli-backend-framework/blob/6137d453d5f2390a77556f3b70c9266c58305334/blibli-backend-framework-api-client/src/main/java/com/blibli/oss/backend/apiclient/aop/ApiClientMethodInterceptor.java#L322) yang tidak me-_return_ Mono dan saya terapkan Mono pada _method_ saya [disini](https://github.com/richard483/raven-api-client/blob/5c267d1c50162d51462785663ed5c0c1afc9272e/src/main/java/com/nephren/raven/apiclient/aop/RavenApiClientMethodInterceptor.java#L224). Tidak berarti _method_ yang terdapat di Api Client Blibli pasti akan _blocking_, namun menurut pemahaman saya karena kita melakukan pemrograman Webflux, seharusnya me-_return_ Mono (mengingat saya masih tahap belajar juga, saya menerima masukan melalui kotak saran di _footer_ terkait hal ini).

---

**References**

- [https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/)
