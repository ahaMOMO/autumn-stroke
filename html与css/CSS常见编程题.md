### 一.实现一个三角形

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        div {
            width: 0;
            height: 0;
            border-width: 5px;
            border-style: solid;
            border-color: red transparent transparent transparent;
        }
    </style>
</head>

<body>
    <div></div>
    <script>
    </script>
</body>

</html>
```

### 二.一个水平居中元素，在按钮点击时候在100ms中往右走100px

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .father {
            position: relative;
            width: 300px;
            height: 300px;
            border: 1px solid red;
        }

        .son{
            position: absolute;
            top: 50%;
            left: 50%;
            margin-left: -50px;
            margin-top: -50px;
            width: 100px;
            height: 100px;
            background: rgb(73, 69, 69);
        }
        .move{
            animation: mymove 100ms;
            -webkit-animation: mymove 100ms;
            animation-fill-mode:forwards;
        }
        @keyframes mymove {
            from {
                margin-left: -50px;
            }

            to {
                margin-left: 50px;
            }
        }

    </style>
</head>

<body>
    <div class="father">
        <button>点击</button>
        <div class="son"></div>
    </div>
    <script>
        let btn = document.getElementsByTagName("button")[0];
        let son = document.getElementsByClassName("son")[0];
        btn.onclick = function () {
            son.className +=" move";
        }
    </script>
</body>

</html>
```

