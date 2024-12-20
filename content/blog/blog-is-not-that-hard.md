+++
title = "Blog Is Not That Hard (블로그 그렇게 어렵지 않다)"
date = "2024-10-23T08:28:30Z"

#
# description is optional
#
# description = "An optional description for SEO. If not provided, an automatically created summary will be used."
description = "Hugo 를 사용하여 블로그를 만들게 된 이야기"

tags = ["blogging", "hugo"]
+++

옛날에는 집 컴퓨터에 Apmsetup 을 통해서 기본적인 APM (Apache, PHP, MySQL) 을 설정
하고, 텍스트큐브 (Textcube) 를 설치해서 블로그를 만들었던 것 같은데 둘다 사라져버린
탓에 지금은 Wordpress 를 호스팅 서비스에 설치하여 사용하는 것이 주류가 된 것 같다.

이후에는 [Jekyll](https://jekyllrb.com/) 과 Github Pages 기능을 통해서 정적 사이트를
만들어 사용했던 것 같다. 그 당시에도 몇번 시도해 봤었는데, Jekyll 이 Ruby 로 만들어져
있어서 Ruby 언어를 따로 알아야 할 것 같은 부담감이 있어서 딥하게는 사용해보진 못했다.

그 이후로 시간이 많이 흘렀다. 시간이 지나가면 지날수록 시간이 사라져만 가고,
귀차니즘 때문에 블로그를 하지 않았다. 오히려 마이크로블로깅 (Instagram, etc) 이 더 좋은
것 같아서 한동한 빠졌었다. Meta 에서 내놓은 Thread 도 했었는데, 처음에는 신기한 사람들
이 가입하고, 뭔가 건설적인(?) 이야기들이 많이 있어서 사용했었는데 지금은 Twitter 그 이상
그 이하도 아닌 것 같다.

어쨌거나, 이것저것 생각을 공유하는 나만의 블로그를 만들어봐야 되겠다 싶어 이것저것 찾아봤는데..

1. 티스토리에서 글을 쓰는 것.
2. [Hugo](https://gohugo.io/) 같은 정적 페이지 생성기를 통해서 Github Pages 에서 호스팅 하는 것.
3. PHP 호스팅에 Wordpress 를 설치해서 블로깅.

이렇게 정리가 되었고, 각 항목별로 찾아보다가 결국엔 (2) 번째의 Hugo 를 사용해서 깃허브 블로그
에 앉게 되었는데, 그 이유는 다음과 같다.

1. 티스토리는 카카오에 흡수가 되어서 Kakao 계정을 필요로 한다. 젤 싫은걸 통하지 않으면 안된다니
포기했다.
2. Cafe24 와 같이 월 몇백원도 안하는 PHP 호스팅에 Wordpress 를 올리는건 쉽다. 문제는 꾸준히 PHP
코드를 수정하고, 최적화를 하는 과정이 필요할 것 같아서 사용하지 않기로 했다.

설정하면서, Hugo 의 [Quick start](https://gohugo.io/getting-started/quick-start/) 가이드를 참고
하니 크게 어려운 것은 없었다. [Themes](https://themes.gohugo.io/) 를 참고하여 나의 용도에 맞는
테마를 선택하여 가이드 대로 설정하면 된다.

배포는 Hugo 의 문서 중 "Hosting and Deployment" 항목의
[Host on Github Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
을 참고하여 배포하면 되는데, 해당 문서는
[Github Workflows](https://docs.github.com/en/actions/writing-workflows) 를 사용하여 배포
하는 것을 권장하고 있다.

근데, Github Workflows 를 설정하는 것은 귀찮은 일이다. (물론 돈을 받는 일을 할때는 CI/CD 같은
것들도 거뜬히 셋업하고 관리했었다. 돈을 받지 않는 일이니 귀찮으 수 밖에 없다.) 그래서 Github
Workflows 를 사용하지 않고 Serving 할 수 있는 방법에 대하여 고민해 보았다.

해결 방법은 간단했는데, Github Pages 에서 서빙하는 디렉토리를 기본 설정이 `/` 에서 `/docs` 으로
변경한 후에 Hugo 의 설정을 변경하여 `docs` 디렉토리에 HTML 을 비롯한 제반적인 설정이 추가되도록
하면 될 것 같았다.

1. Github Pages Repository 의 Settings 의 "Pages" 메뉴에서 Branch 항목의 디렉토리를 `/docs` 으로
변경한다.
2. `hugo.toml` 파일의 `publishDir = 'docs'` 으로 설정하여 `docs` 디렉토리로 변경한다.
3. 만약 `$ hugo` 혹은 `$ hugo server` 를 실행하여 `public` 디렉토리가 존재하는 경우, 해당 디렉토리
를 삭제한다. (꼭 삭제할 필요는 없지만, 기존의 찌꺼기 파일들이 존재하는 것을 고민해보면 삭제하는
것이 좋을 것 같다.)

이렇게 설정하면 글 작성 이후에 `$ hugo` 를 실행하고, `docs` 의 부산물을 git 에 추가하여 push 하면
된다.

당분간은 이렇게 써보고, 필요하면 Github Workflows 를 통해서 CI/CD 를 적용해보아야 되겠다.
(그게 언제가 될지는 모르겠지만?)

끝.
