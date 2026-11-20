# KhelaStream MCP Server

এটি KhelaStream-এর Firebase Realtime Database-এর জন্য একটি Remote MCP Server starter।
Claude এবং MCP-compatible ChatGPT integration-এ একই server ব্যবহার করা যাবে।

## কী করতে পারবে

- ম্যাচের তালিকা দেখা ও নাম দিয়ে খোঁজা
- ম্যাচ যোগ করা, নির্দিষ্ট তথ্য আপডেট করা
- নিশ্চিতকরণ phrase ছাড়া ম্যাচ delete হবে না
- কোন ম্যাচে Server 1/2/3 সেট করা আছে কি না দেখা

**নিরাপত্তা/সীমা:** AI-কে M3U8/stream URL খুঁজে দেওয়া বা তৈরি করার tool দেওয়া হয়নি। `streams` path থেকে URL-ও ফেরত দেয় না। Stream URL শুধু আপনার নিজের/অনুমোদিত উৎস থেকে Admin Panel-এ হাতে যোগ করবেন।

## প্রয়োজন

- Node.js 20+
- Firebase Realtime Database access
- Firebase service account (server-side only)
- HTTPS hosting (যেমন Render/Railway/Fly.io বা নিজের VPS)

## 1. Firebase service account তৈরি

Firebase Console → Project settings → Service accounts → Generate new private key।
ডাউনলোড করা JSON কখনো Blogger theme, GitHub public repo, বা browser code-এ রাখবেন না।
এটি hosting-এর secret/environment variable-এ দিন: `FIREBASE_SERVICE_ACCOUNT_JSON`।

Database URL:
`https://khelastream-xyz-default-rtdb.firebaseio.com`

## 2. Secret তৈরি

একটি লম্বা random secret তৈরি করে `MCP_API_TOKEN` হিসেবে hosting-এ রাখুন।
একই secret Claude-এর connector-এ Bearer token হিসেবে ব্যবহার হবে।
কাউকে token বা service-account JSON দেবেন না।

## 3. Deploy

এই folder GitHub private repository-তে push করে Node service হিসেবে deploy করুন।
Build command: `npm install`
Start command: `npm start`
Environment variables:
- `MCP_API_TOKEN`
- `FIREBASE_DATABASE_URL`
- `FIREBASE_SERVICE_ACCOUNT_JSON`
- `PORT` (hosting নিজে দিলে সেটিই ব্যবহার হবে)

Deploy হলে:
- Health: `https://YOUR-HOST/health`
- MCP: `https://YOUR-HOST/mcp`

MCP endpoint অবশ্যই public HTTPS port 443 দিয়ে reachable হতে হবে।

## 4. Claude-এ যোগ

Claude → Customize → Connectors → Add custom connector → Web।
Name: `KhelaStream`
URL: `https://YOUR-HOST/mcp`
Authentication: Bearer token / Request header (Claude UI-তে যেভাবে দেখায়)
Header name: `Authorization`
Value: `Bearer YOUR_MCP_API_TOKEN`

Claude-এর UI যদি fixed request header না দেখায় বা OAuth চায়, সেটিংসের auth detection অনুসরণ করুন; server-এ OAuth চালু নেই।

## 5. MCP client দিয়ে পরীক্ষা

`Authorization: Bearer YOUR_MCP_API_TOKEN` header ছাড়া MCP endpoint কাজ করবে না।
প্রথমে Claude connector-এ connect করে `list_matches` চালিয়ে পরীক্ষা করুন।

## Available tools

- `list_matches`
- `get_match`
- `find_match_by_name`
- `add_match`
- `update_match`
- `delete_match` (confirm = `DELETE MATCH` বাধ্যতামূলক)
- `list_stream_status`

## গুরুত্বপূর্ণ

এই starter আপনার Firebase-এ সরাসরি কাজ করার জন্য তৈরি, তবে deploy করার আগে Firebase project/database path মিলিয়ে নিন।
Firebase Admin SDK database rules bypass করে; তাই MCP token-টি খুব গোপন রাখুন। হারিয়ে গেলে সঙ্গে সঙ্গে hosting-এ token rotate করুন।
এটি একটি starter implementation; production-এ ব্যবহারের আগে access logs, rate limiting, backup এবং পৃথক read/write token যোগ করার পরামর্শ দেওয়া হয়।
