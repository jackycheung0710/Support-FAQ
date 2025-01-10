### Typora激活

Typora\resources\page-dist\static\js\LicenseIndex.180dd4c7.b98d5f1f.chunk.js文件：

```javascript
e.hasActivated="true"==e.hasActivated 
替换为 
e.hasActivated="true"=="true"
```

Typora\resources\page-dist\license.html文件：

```html
</body></html>
替换为 
</body><script>window.οnlοad=function(){setTimeout(()=>{window.close();},5);}</script></html>
```

Typora\resources\locales\zh-Hans.lproj\Panel.json文件：

```json
"UNREGISTERED":"未激活"，
替换为：
"UNREGISTERED":" "
```

