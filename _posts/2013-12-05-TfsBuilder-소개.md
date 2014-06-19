---
layout: post
title: TfsBuilder 소개
tags : [Miscellaneous]
---
{% include JB/setup %}


[Visual Studio Online]에서는 무료로 월 60분의 빌드서비스를 제공하고 있습니다. CI(Continuous Integration)서버를 직접 구축하지 않고 이용할 수 있다는 것이 큰 매리트인 것 같습니다. 무료 월 60분이 좀 적은 시간인 것 같긴 하지만 그래도 이게 어딘가하는 생각이 드네요. 부족 분에 대해서는 분 당 30원의 이용금액을 지불하고 이용할 수 있다고 합니다.([가격정책] 참고)

이렇게 말하고 보니 뭐 마소 직원인 것 같아 보이는데 전혀 상관없는 사람입니다. 이 글을 쓰는 목적은 [TfsBuilder]를 소개하기 위함입니다. Visual Studio Online에서는 자체 소스리포지토리를 이용할 경우는 CI 빌드를 제공합니다. 즉, 소스를 중앙리포지토리로 push하면 이것을 신호로 서버에서는 빌드작업을 하게 됩니다. 이 빌드작업에 배포작업까지 끼우게 되면 소스 push로만 빌드에서 배포까지 이루어지는 것입니다.

그러나, Visual Studio Online에서 만약 GitHub와 같은 외부 소스 리포지토리를 이용하게 되면, CI빌드를 제공하지 않고, 스케줄 빌드 혹은 매뉴얼(직접)빌드만 제공합니다. 그래서 Visual Studio Online에 대한 빌드 서비스훅을 만들어 보았습니다.

소스와 사용법에 대해서는 GitHub프로젝트 [TfsBuilder]에 있습니다.

[Visual Studio Online]: http://www.visualstudio.com/
[가격정책]: http://www.windowsazure.com/ko-kr/pricing/details/visual-studio-online/
[TfsBuilder]: https://github.com/jwChung/TfsBuilder