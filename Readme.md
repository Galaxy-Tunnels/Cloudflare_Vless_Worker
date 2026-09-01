# Galaxy VProxy Worker – ပြင်ဆင်ထားသော Version

ဒီ package ထဲက `worker.js` သည် မူရင်း Worker ကို အခြေခံပြီး အောက်ပါအချက်များကို ပြင်ဆင်ထားသော version ဖြစ်သည်။

- `PROXY_LIST_URL` ကို `https://gprox-galaxy.github.io/PROXYIP.txt` သို့ ပြောင်းထားသည်။
- WebSocket binary frame ကို `ArrayBuffer` အဖြစ် explicit သတ်မှတ်ထားသည်။
- WebSocket proxying အတွက် half-open mode ထည့်ထားသည်။
- WebSocket write များကို await လုပ်ပြီး writer lock ကို `finally` ဖြင့် release လုပ်သည်။
- Text WebSocket frame များကို reject လုပ်သည်။
- gRPC input size limit ထည့်ထားသည်။
- VLESS option, port, IPv4, domain နှင့် IPv6 buffer boundary check များ ထည့်ထားသည်။
- Proxy list မှ invalid/အလွန်ရှည်သော hostname များကို filter လုပ်ထားသည်။
- WebSocket/gRPC outbound handler များကို await လုပ်ထားသည်။

## Cloudflare Worker ဖြင့် Deploy လုပ်ခြင်း

1. Cloudflare account တွင် Workers & Pages > Create application > Worker ကိုရွေးပါ။
2. `worker.js` ကို upload/copy လုပ်ပါ။
3. `wrangler.toml` ကို project root တွင်ထားပါ။
4. Dashboard ရှိ Settings > Variables and Secrets မှ အောက်ပါ variable များ ထည့်ပါ။

| Variable | Type | တန်ဖိုး |
|---|---|---|
| `UUID` | Secret | ကိုယ်ပိုင် VLESS UUID |
| `WS_PATH` | Secret သို့မဟုတ် variable | ခန့်မှန်းရခက်သော path ဥပမာ `/x7m9-galaxy-2026` |
| `PROXY_LIST_URL` | Variable | `https://gprox-galaxy.github.io/PROXYIP.txt` |
| `DNS_RESOLVER_URL` | Variable | `https://cloudflare-dns.com/dns-query` |

`UUID` ကို source code သို့မဟုတ် Git repository ထဲ မထည့်ပါနှင့်။ Wrangler CLI သုံးမည်ဆိုလျှင် `npx wrangler secret put UUID` ဖြင့် ထည့်ပါ။ `WS_PATH` ကိုလည်း public repository ထဲ မရေးဘဲ dashboard variable အဖြစ် ထည့်သင့်သည်။

## Cloudflare Pages အသုံးပြုမည်ဆိုလျှင်

Pages သည် static HTML hosting နှင့် Workers Functions ကို ပေါင်းစပ်အသုံးပြုနိုင်သည်။ Static landing page ကို Pages တွင်ထားပြီး proxy Worker ကို သီးခြား Worker အဖြစ်ထားခြင်းက အလွယ်ဆုံးနှင့် စောင့်ကြည့်ပြင်ဆင်ရန် အဆင်ပြေဆုံးဖြစ်သည်။ Pages Functions အဖြစ်အသုံးပြုလိုပါက `worker.js` ကို Pages Functions ရဲ့ `_worker.js` format နှင့် project structure အတိုင်း ပြောင်းရနိုင်သဖြင့် Worker deployment ကို ဦးစားပေးအကြံပြုသည်။

## Proxy list အကြောင်း

`https://gprox-galaxy.github.io/PROXYIP.txt` သည် public URL ဖြစ်သဖြင့် file ထဲရှိ host များကို မည်သူမဆို ပြောင်းလဲနိုင်သည်။ ထိုကြောင့် commercial product တွင် ၎င်း URL ကို တစ်ခုတည်းသော trust source အဖြစ် မထားသင့်ပါ။ ကိုယ်ပိုင် repository, signed list, hostname allowlist, health check နှင့် emergency disable variable ထည့်သင့်သည်။

## မတင်မီ စမ်းသပ်ရန်

```bash
node --check worker.js
npx wrangler deploy
```

အောက်ပါ test များကို Cloudflare preview နှင့် production domain နှစ်ခုလုံးတွင် စမ်းပါ။ Browser မှ normal HTTPS request, မှားယွင်းသော WebSocket path, မှားယွင်းသော UUID, WebSocket binary frame, gRPC client, DNS UDP query, destination connection failure, reconnect နှင့် long-lived connection တို့ ဖြစ်သည်။

## လက်ရှိကန့်သတ်ချက်များ

ဒီ version သည် full commercial VPN control plane မဟုတ်သေးပါ။ Per-user quota, billing, admin dashboard, user revoke API, detailed usage accounting, abuse detection, upstream health scoring နှင့် automated rollback မပါသေးပါ။ Public service အဖြစ် ဖွင့်မည်ဆိုလျှင် အဆိုပါအပိုင်းများကို ထပ်မံတည်ဆောက်ရန်လိုသည်။
