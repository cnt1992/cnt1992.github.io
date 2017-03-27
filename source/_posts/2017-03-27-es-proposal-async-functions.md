title: ���롿ES���飺async����
date: 2017-03-27 17:16:56
categories: ����
tags: ES7 async await
description: ��Ҫ�������첽��������ʷ��ͬʱ�Ƽ�ES7��async�������Ͼ�ES6��generator��yieldֻ��һ������
---

`Async`������`Brian Terlson`�����`ECMAScript�᰸`��Ŀǰ����`stage 3(candidate ��ѡ)`�׶Ρ�

| ����ע��ECMAScript�᰸�������׶Σ���ͼ(Sketch) -> �᰸(Proposal) -> �淶(Standard)������`Async`�ܴ��ϣ���ܳ�Ϊ�淶.

�ڿ�ʼ����`async`����֮ǰ�������Ƚ�һ�����ͨ�����`Promises`��`generators`��**������ͬ���Ĵ���**�������첽������

## ͨ�� Promises �� generators д�첽����

���ڶ���������첽�ĺ����У�`Promises`����Ϊ`ES6`��һ���֣����Խ��Խ�ܻ�ӭ��һ�����Ӿ��ǿͻ��˵�`fetch API`���������������ȡ���ݴӶ����`XMLHttpRequest`��ʹ��`fetch`������������������

```javascript
function fetchJson(url) {
    return fetch(url)
        .then(request => request.text())
        .then(text => {
            return JSON.parse(text);
        })
        .catch(error => {
            console.log(`ERROR: ${error.stack}`);
        });
}
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

[co](https://github.com/tj/co)��ͨ��ʹ��`Promises`��`generators`���ô��뿴������ͬ���������������`co`��д�������������ӣ�

```javascript
const fetchJson = co.wrap(function* (url) {
    try {
        let request = yield fetch(url);
        let text = yield request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
});
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

ÿһ�λص�(һ��`generator`����)���ᴫ��һ��`Promises`����`co`��Ȼ��ص�������ͣ״̬��һ��`Promise`ִ����ϣ�`co`�ͻᴥ���ص������`Promise`����`fulfilled(�ɹ�)`��`yield`���سɹ�״̬��ֵ���������`rejected(ʧ��)`��`yield`�ͻ��׳�ʧ�ܵĴ������⣬`co`�Ὣ���صĻص���װ��һ��`Promise`(��`then()`����;һ��)��

## Async ����

�����������`Async`�����Ļ����÷�ʵ�����£�

```javascript
async function fetchJson(url) {
    try {
        let request = await fetch(url);
        let text = await request.text();
        return JSON.parse(text);
    }
    catch (error) {
        console.log(`ERROR: ${error.stack}`);
    }
}
fetchJson('http://example.com/some_file.json').then(obj => console.log(obj));
```

��ʵ���ϣ�`async`��������`generators`�����ǲ���ת����`generatos`����.

Internally, async functions work much like generators, but they are not translated to generator functions.

### Async ����������һЩ�÷�

���ڴ������µ�`async`�����ı��壺

- Async ���������� `async function foo() {}`
- Async �������ʽ�� `const foo = async function() {}`
- Async �������壺 `let obj = { async foo() {} }`
- Async ��ͷ������`const foo = async () => {};`

## �����Ķ�

[Async Functions](https://github.com/tc39/ecmascript-asyncawait)(Brian Terlson)
[Simplifying asynchronous computations via generators](http://exploringjs.com/es6/ch_generators.html#sec_co-library)(��Exploring ES6�����½�)