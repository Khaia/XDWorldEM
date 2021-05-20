웹 어플리케이션을 위한 지구본 기반 WebGL GIS 엔진 활용법에 대해 소개합니다.


## Getting started


### 1. 엔진 파일 로드
> 엔진 파일은 총 3개 파일로 구성됩니다.
> * XDWorldEM.asm.js
> * XDWorldEM.html.mem
> * XDWorldEM.js
> 
> 엔진 파일의 로드 순서는 다음과 같습니다.
> 
> <img src="http://api.xdmap.com/Manual/custom/img/initEnvironment/01.png" align="center"></img>
> 
> 엔진 파일이 모두 로드 되면 API 호출이 가능한 전역 변수 Module을 사용하실 수 있습니다.
> 
> <img src="http://api.xdmap.com/Manual/custom/img/initEnvironment/02.png" align="center"></img>


### 2. Canvas 연결하기
> Module 객체가 생성되면 지도를 출력하는 canvas와 Module 객체를 연결합니다.
> 
> Module.canvas 속성에 canvas 엘리먼트를 대입하여 지도를 출력할 canvas를 지정할 수 있습니다.
> ```
> var Module = {
> 	postRun : [init],
> 	canvas : document.getElementById("canvas")
> };
> ```
> 
> 동적으로 canvas를 생성하여 Module 객체와 연결하는 방식도 가능합니다.
>
> ```
> var Module = {
> 	TOTAL_MEMORY: 256*1024*1024,
> 	postRun : [init],
> 	canvas : (function() {
> 	
> 		// Canvas 엘리먼트 생성
> 		var canvas = document.createElement('canvas');
> 		
> 		// Canvas id, Width, Height 속성 설정
> 		canvas.id = "canvas";
> 		canvas.width="calc(100%)";
> 		canvas.height="100%";
> 		
> 		// Canvas 스타일 설정
> 		canvas.style.position = "fixed";
> 		canvas.style.top = "0px";
> 		canvas.style.left = "0px";
> 		
> 		// 생성한 Canvas 엘리먼트를 document에 추가
> 		document.body.appendChild(canvas);
> 		
> 		// Module.canvas 속성으로 canvas 엘리먼트 반환
> 		return canvas;
> 	})()
> };
> ```


### 3. 엔진 초기화 함수 실행
> 엔진 파일이 모두 로드되면 엔진 환경 초기화 함수를 연결합니다.
> 
> 초기화 함수는 Module 객체의 postRun 속성 대입하여 지정합니다.
> ```
> var Module = {
> 	postRun : [init],
> 	canvas : document.getElementById("canvas")
> };
> 
> // 초기화 함수
> function init( ) {
> 
> 	// 엔진 초기화 API 호출(필수)
> 	Module.Start(window.innerWidth, window.innerHeight);
> }
> ```
> 
> 초기화 함수가 실행되는 시점은 엔진 파일이 모두 로드된 후 엔진 메인 루프가 실행되기 이전 입니다.
> <img src="http://api.xdmap.com/Manual/custom/img/initEnvironment/03.png" align="center"></img>


### 3. 엔진 구동
> Module.Start API를 호출하면 엔진 렌더링이 시작됩니다.
> ```
> function init( ) {
>	
>	// 엔진 초기화 API 호출(필수)
>	Module.Start(window.innerWidth, window.innerHeight);
>}
> ```
> <img src="http://api.xdmap.com/Manual/custom/img/initEnvironment/04.png" align="center"></img>


## 배포 엔진 파일 구성
> <img src="http://api.xdmap.com/Manual/custom/img/initEnvironment_sample/00.png" align="center"></img>
> 
> 엔진 파일은 버전 별로 엔진파일 / 실행 스크립트 / 기본 HTML 페이지를 함께 배포합니다.
> 
> index.html 페이지에서 init.js 스크립트를 호출하고, init.js 스크립트는 엔진 로드, Canvas 설정, 초기화 함수 설정, 지도 데이터 로드 과정을 실행합니다.
> 
> *[HTML] index.html*
> ```
> <!doctype html>
> <html>
> <head>
> 	<title>[EGIS] Init
> </head>
> <body>
> 	<script>
> 		var initScript = document.createElement('script');
> 		initScript.src = "./js/init.js";
> 		document.body.appendChild(initScript);
> 	</script>
> </body>
> </html>
> ```
> 
> *[JavaScript] init.js*
> ```
> // 엔진 로드 후 실행할 초기화 함수(Module.postRun)
> function init() {
> 
> 	// 엔진 초기화 API 호출(필수)
> 	Module.SetAPIKey("767B7ADF-10BA-3D86-AB7E-02816B5B92E9");	// 이 곳에 배포된 브이월드 API 키를 입력하세요.
> 	Module.Start(window.innerWidth, window.innerHeight);
> }
> 
> // 엔진 파일 로드
> ;(function(){   	
> 
> 	// 1. XDWorldEM.asm.js 파일 로드
> 	var file = "./js/XDWorldEM.asm.js";
> 	
> 	var xhr = new XMLHttpRequest();
> 	xhr.open('GET', file, true);
> 	xhr.onload = function() {
	
> 		var script = document.createElement('script');
> 		script.innerHTML = xhr.responseText;
> 		document.body.appendChild(script);
> 		
> 		// 2. XDWorldEM.html.mem 파일 로드
> 		setTimeout(function() {
> 			(function() {
> 				var memoryInitializer = "./js/XDWorldEM.html.mem";
> 				var xhr = Module['memoryInitializerRequest'] = new XMLHttpRequest();
>         			xhr.open('GET', memoryInitializer, true);
> 					xhr.responseType = 'arraybuffer';
> 					xhr.onload =  function(){
> 						
> 						// 3. XDWorldEM.js 파일 로드
> 						var url = "./js/XDWorldEM.js";
> 						var xhr = new XMLHttpRequest();
> 						xhr.open('GET',url , true);
> 						xhr.onload = function(){
> 							var script = document.createElement('script');
> 							script.innerHTML = xhr.responseText;
> 							document.body.appendChild(script);
> 						};
> 						xhr.send(null);
> 					}
> 					xhr.send(null);
> 				})();
> 			}, 1);
> 		};
> 		xhr.send(null);
> 	}
> )();
> 
> var Module = {
>	TOTAL_MEMORY: 256*1024*1024,
> 	postRun: [init],
> 	canvas: (function() {
> 		
> 		// Canvas 엘리먼트 생성
> 		var canvas = document.createElement('canvas');
> 		
> 		// Canvas id, Width, height 설정
> 		canvas.id = "canvas";
> 		canvas.width="calc(100%)";
> 		canvas.height="100%";
> 		
> 		// Canvas 스타일 설정
> 		canvas.style.position = "fixed";
> 		canvas.style.top = "0px";
> 		canvas.style.left = "0px";
> 
> 		// 생성한 Canvas 엘리먼트를 body에 추가합니다.
> 		document.body.appendChild(canvas);
> 		return canvas;
> })()
> };
> ```

## Demo
http://www.dtwincloud.com:8080/dt_reference/code/main.do?id=start

## 최신 Release 정보 (0.1.0 / 2021-05-17)
* 0.1.0 버전 Release
