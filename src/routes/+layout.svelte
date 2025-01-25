<script>
    import { onMount } from "svelte";
    import "../app.css";
    import { injectAnalytics } from "@vercel/analytics/sveltekit";

    onMount(() => {
        injectAnalytics();
    });
</script>

<svelte:head>
    <title>인생 달력 - 삶은 zoom in&out이다.</title>
    <meta name="naver-site-verification" content="5b72ab228d49e1d1b0794f740449af76dfc399f3" />
    <!-- 메타 정보 (일반) -->
    <meta
        name="description"
        content="여러분 삶을 줌인해서 하루하루를 꾸준히 기록하고, 줌아웃해서 '주-년-삶'의 기록을 거시적으로 바라보세요! 광고도, 회원가입도, 추적도 없으니 부담 없이 시작해보세요.(중요한 건 당신의 의지!)"
    />
    <meta name="keywords" content="인생 달력, 인생 테이블, 습관 추적기, 라이프 캘린더, 생산성, 자기계발" />

    <!-- 오픈 그래프 (SNS 미리보기용) -->
    <meta property="og:title" content="인생 달력 - 색으로 기록하는 나의 하루" />
    <meta
        property="og:description"
        content="여러분 삶을 줌인해서 하루하루를 꾸준히 기록하고, 줌아웃해서 '주-년-삶'의 기록을 거시적으로 바라보세요! 광고도, 회원가입도, 추적도 없으니 부담 없이 시작해보세요.(중요한 건 당신의 의지!)"
    />
    <meta property="og:type" content="website" />
    <meta property="og:url" content="https://life.mmeme.org" />
    <!-- <meta property="og:image" content={ogImageUrl.toString()} />
    <meta property="og:image:width" content="1200" />
    <meta property="og:image:height" content="630" /> -->

    <!-- 트위터 카드 -->
    <meta name="twitter:card" content="summary" />
    <meta name="twitter:title" content="인생 달력 - 색으로 기록하는 나의 하루" />
    <meta
        name="twitter:description"
        content="여러분 삶을 줌인해서 하루하루를 꾸준히 기록하고, 줌아웃해서 '주-년-삶'의 기록을 거시적으로 바라보세요! 광고도, 회원가입도, 추적도 없으니 부담 없이 시작해보세요.(중요한 건 당신의 의지!)"
    />
</svelte:head>

<slot />
