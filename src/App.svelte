<script>
  my.getAuthCode({
    scopes: ["auth_base", "USER_ID"],
    success: (res) => {
      my.alert({
        content: res.authCode,
      });
    },
    fail: (res) => {
      console.log(res.authErrorScopes);
    },
  });
</script>

<main>
  <div class="auth-card animate-enter">
    <div class="auth-header">
      <!-- Simple abstract logo -->
      <svg
        width="48"
        height="48"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg"
        style="margin-bottom: 1rem;"
      >
        <rect width="24" height="24" rx="12" fill="var(--primary-light)" />
        <path
          d="M12 7C9.23858 7 7 9.23858 7 12C7 14.7614 9.23858 17 12 17C14.7614 17 17 14.7614 17 12"
          stroke="var(--primary)"
          stroke-width="2"
          stroke-linecap="round"
        />
        <circle cx="12" cy="12" r="3" fill="var(--primary)" />
      </svg>

      <h1 class="auth-title">Welcome Back</h1>
      <p class="auth-subtitle">Sign in to access your workspace</p>
    </div>

    <div class="auth-actions">
      <button
        class="btn btn-primary"
        type="button"
        onClick={() =>
          my.getAuthCode({
            scopes: ["auth_base", "USER_ID"],
            success: (res) => {
              my.alert({
                content: res.authCode,
              });
            },
            fail: (res) => {
              console.log(res.authErrorScopes);
            },
          })}
      >
        Continue
      </button>
    </div>
  </div>
</main>
