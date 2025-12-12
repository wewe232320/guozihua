# guozihua
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unity WebGL 演示</title>
    <style>
        * {
            box-sizing: border-box;
        }
        
        body { 
            margin: 0; 
            padding: 20px; 
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            font-family: Arial, sans-serif, "Microsoft YaHei";
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }
        
        .container {
            background: white;
            border-radius: 10px;
            padding: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            text-align: center;
            max-width: 800px;
            width: 100%;
        }
        
        #unity-container { 
            width: 800px; 
            height: 600px; 
            margin: 20px auto;
            position: relative;
        }
        
        #unity-canvas { 
            width: 100%; 
            height: 100%; 
            background: #231F20; 
            border-radius: 5px;
        }
        
        #unity-loading { 
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 18px;
            text-align: center;
            z-index: 10;
            display: block;
            background: rgba(0,0,0,0.7);
            padding: 15px 30px;
            border-radius: 10px;
        }
        
        h1 { 
            color: white; 
            margin-bottom: 10px; 
            text-shadow: 2px 2px 4px rgba(0,0,0,0.5);
        }
        
        .instructions {
            background: rgba(255,255,255,0.1);
            padding: 15px;
            border-radius: 8px;
            margin: 20px 0;
            color: white;
            backdrop-filter: blur(10px);
            max-width: 800px;
            width: 100%;
        }
        
        .instructions a {
            color: #ffd700;
            text-decoration: none;
            font-weight: bold;
        }
        
        .instructions a:hover {
            text-decoration: underline;
        }
        
        /* 移动端适配 */
        @media (max-width: 850px) {
            #unity-container { 
                width: 100% !important; 
                height: 400px !important;
            }
        }
        
        @media (max-width: 600px) {
            body { 
                padding: 10px; 
            }
            
            .container {
                padding: 15px;
            }
            
            #unity-container { 
                height: 300px !important;
            }
            
            h1 {
                font-size: 1.5em;
            }
        }
        
        /* 错误提示样式 */
        .error-message {
            display: none;
            background: #ff6b6b;
            color: white;
            padding: 15px;
            border-radius: 5px;
            margin: 10px 0;
            text-align: center;
        }
        
        /* 操作说明 */
        .controls {
            background: #f8f9fa;
            border-left: 4px solid #667eea;
            padding: 10px 15px;
            margin: 15px 0;
            text-align: left;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <h1>🎮 Unity WebGL 演示</h1>
    
    <div class="instructions">
        <p>这是一个简单的Unity WebGL应用示例。立方体会自动旋转。</p >
        <p>项目地址：<a href=" " target="_blank">GitHub</a ></p >
    </div>
    
    <div class="error-message" id="error-message"></div>
    
    <div class="container">
        <h2>3D旋转立方体演示</h2>
        
        <div class="controls">
            <strong>操作说明：</strong>
            <ul style="margin: 5px 0; padding-left: 20px;">
                <li>鼠标左键拖拽：旋转视角</li>
                <li>鼠标滚轮：缩放</li>
                <li>右键拖拽：平移视角</li>
            </ul>
        </div>
        
        <div id="unity-container">
            <canvas id="unity-canvas" width="800" height="600"></canvas>
            <div id="unity-loading">正在加载Unity应用...<br><span id="progress">0%</span></div>
        </div>
        
        <p style="color: #666; margin-top: 15px; font-size: 0.9em;">
            如果加载失败，请确保浏览器支持WebGL并刷新页面
        </p >
    </div>

    <script>
        // 配置Unity构建文件
        var buildUrl = "Build";
        var projectName = "WebGLBuild"; // 修改为你的项目名
        
        // 尝试不同的加载器文件名
        var loaderFiles = [
            projectName + ".loader.js",
            projectName + ".js",
            "loader.js"
        ];
        
        var config = {
            dataUrl: buildUrl + "/" + projectName + ".data",
            frameworkUrl: buildUrl + "/" + projectName + ".framework.js",
            codeUrl: buildUrl + "/" + projectName + ".wasm",
            streamingAssetsUrl: "StreamingAssets",
            companyName: "Unity WebGL Demo",
            productName: "3D Cube Demo",
            productVersion: "1.0",
        };
        
        var container = document.querySelector("#unity-container");
        var canvas = document.querySelector("#unity-canvas");
        var loading = document.querySelector("#unity-loading");
        var progressSpan = document.querySelector("#progress");
        var errorMessage = document.querySelector("#error-message");
        
        // 移动端检测和适配
        function isMobile() {
            return /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
        }
        
        // 检查WebGL支持
        function checkWebGLSupport() {
            try {
                if (!window.WebGLRenderingContext) {
                    return false;
                }
                
                var canvas = document.createElement('canvas');
                var contexts = ["webgl", "experimental-webgl", "webkit-3d", "moz-webgl"];
                var gl = false;
                
                for (var i = 0; i < contexts.length; i++) {
                    try {
                        gl = canvas.getContext(contexts[i]);
                        if (gl && typeof gl.getParameter === "function") {
                            return true;
                        }
                    } catch(e) {}
                }
                
                return !!gl;
            } catch(e) {
                return false;
            }
        }
        
        // 显示错误
        function showError(message) {
            errorMessage.textContent = "错误: " + message;
            errorMessage.style.display = "block";
            loading.style.display = "none";
            console.error(message);
        }
        
        // 检查WebGL支持
        if (!checkWebGLSupport()) {
            showError("您的浏览器不支持WebGL，请使用最新版本的Chrome、Firefox或Edge浏览器");
        }
        
        // 移动端适配
        if (isMobile()) {
            container.className = "unity-mobile";
            canvas.style.width = "100%";
            canvas.style.height = "100%";
            
            // 移动端建议
            var instructions = document.querySelector('.instructions');
            instructions.innerHTML += "<p style='color: #ffd700; margin-top: 10px;'>💡 移动端建议：横屏模式下体验更佳</p >";
        }
        
        // 加载Unity应用
        function loadUnity() {
            loading.style.display = "block";
            
            // 尝试加载加载器
            var script = document.createElement("script");
            var loaderLoaded = false;
            
            // 尝试不同的加载器文件
            function tryLoadLoader(index) {
                if (index >= loaderFiles.length) {
                    showError("无法找到Unity加载器文件，请确保构建文件正确放置在Build目录中");
                    return;
                }
                
                script.onerror = function() {
                    console.log("尝试加载 " + loaderFiles[index] + " 失败");
                    tryLoadLoader(index + 1);
                };
                
                script.onload = function() {
                    loaderLoaded = true;
                    console.log("成功加载: " + loaderFiles[index]);
                    
                    // 创建Unity实例
                    if (typeof createUnityInstance !== 'function') {
                        showError("Unity加载器未正确初始化");
                        return;
                    }
                    
                    createUnityInstance(canvas, config, function(progress) {
                        var percentage = Math.round(progress * 100);
                        progressSpan.textContent = percentage + "%";
                        loading.innerHTML = "正在加载Unity应用...<br><span id='progress'>" + percentage + "%</span>";
                    }).then(function(unityInstance) {
                        console.log("Unity应用加载完成！");
                        loading.style.display = "none";
                        
                        // 添加一些交互示例
                        window.unityInstance = unityInstance;
                    }).catch(function(message) {
                        showError("Unity应用加载失败: " + message);
                    });
                };
                
                script.src = buildUrl + "/" + loaderFiles[index];
                document.body.appendChild(script);
            }
            
            tryLoadLoader(0);
        }
        
        // 页面加载完成后启动
        window.addEventListener('DOMContentLoaded', function() {
            // 延迟加载，确保DOM完全加载
            setTimeout(loadUnity, 500);
        });
        
        // 处理窗口大小变化
        window.addEventListener('resize', function() {
            if (isMobile()) {
                canvas.style.width = "100%";
                canvas.style.height = "100%";
            }
        });
    </script>
</body>
</html>
