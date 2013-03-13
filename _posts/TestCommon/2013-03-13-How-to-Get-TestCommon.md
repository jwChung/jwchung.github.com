---
layout: post
title: How to Get TestCommon
tags : [TestCommon]
ccl: ko
date: 2013-03-13 11:34:00 UTC
---
{% include JB/setup %}

<style>
.nuget-badge code
{
	-moz-border-radius: 5px;
	-webkit-border-radius: 5px;
	background-color: #202020;
	border: 4px solid #c0c0c0;
	border-radius: 5px;
	box-shadow: 2px 2px 3px #6e6e6e;
	color: #e2e2e2;
	display: block;
	font: 1.5em 'andale mono', 'lucida console', monospace;
	line-height: 1.5em;
	overflow: auto;
	padding: 15px;
}
</style>

### Install
[TestCommon]은 [NuGet]에 등록되어있다. [Package Manager Console]에서 아래와 같은 명령으로 [TestCommon]을 설치할 수 있다.

<div class="nuget-badge">
    <p>
        <code>PM&gt; Install-Package TestCommon</code>
    </p>
</div>

### 소스코드
[TestCommon]은 [BSD라이센스]하에 배포되는 오픈소스이며, [Github][TestCommon]에 소스가 등록되어있다.
아래와 같이 [Git]의 clone명령을 통해 [TestCommon]의 [Git]리퍼지토리를 로컬에 복제할 수 있다.

```
git clone https://github.com/jwChung/TestCommon.git
```
또한 [TestCommon Tags]페이지에서는 버전별 소스를 압축파일 형태로 제공한다.

[TestCommon]: <https://github.com/jwChung/TestCommon>
[NuGet]: <http://nuget.org/>
[Package Manager Console]: <http://docs.nuget.org/docs/start-here/using-the-package-manager-console>
[BSD라이센스]: <http://www.opensource.org/licenses/bsd-license.php>
[Git]: <http://git-scm.com/>
[TestCommon Tags]: <https://github.com/jwChung/TestCommon/tags>