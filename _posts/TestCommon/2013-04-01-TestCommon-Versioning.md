---
layout: post
title: TestCommon Versioning
tags : [TestCommon]
ccl: ko
date : 2013-04-01 08:23:00 UTC
---
{% include JB/setup %}

TestCommon의 버전은 Semantic Versioning을 따른다.
자세한 내용은 [http://semver.org/](http://semver.org/)에서 볼 수 있는데,
주요 내용을 정리하면 아래와 같다.

*   **버전번호**는 아래와 같이 구성됨

        <major>.<minor>.<patch>

*   **major 버전번호** 증가  
하위 버전의 호환을 보장하지 못하는 모든 변경(minor와 patch버전 0으로 리셋)

*   **minor 버전번호** 증가  
하위 버전의 호환을 보장하는 새로운 기능 추가(patch버전 0으로 리셋)

*   **patch 버전번호** 증가  
하위 버전의 호환을 보장하는 버그 수정

이를 토대로 TestCommon의 최신버전 **v3.1.3**을 살펴보면,
3번의 하위버전 호환이 안되는 변경이 이루어졌으며,
1번의 새로운 기능 추가, 3번의 버그가 수정이 되었음을 의미한다.
자세한 수정 내용은 [TestCommon의 Commits](https://github.com/jwChung/TestCommon/commits/master)을 통해 살펴볼 수 있는데,
커밋 메세지가
\* 혹은 [patch]로 시작하는 경우는 patch 버전번호를 증가시키는 변경이 이루어졌음을 의미하고,
\- 혹은 [minor]로 시작하는 경우는 minor 버전번호를,
\+ 혹은 [major]로 시작하는 경우는 major 버전번호를 증가시키는 변경이 이루어졌음을 의미한다.

<!-- break -->