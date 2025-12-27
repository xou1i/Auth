<script>
  // @ts-nocheck

  let isLoading = false;
  let isPaying = false;
  let user = null;
  let token = null;
  let error = null;

  function loginWithSuperQi() {
    isLoading = true;
    error = null;

    // SuperQi Auth
    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            code: res.authCode,
          }),
        })
          .then((response) => {
            if (!response.ok) throw new Error("Login failed");
            return response.json();
          })
          .then((data) => {
            user = data.record;
            token = data.token;
          })
          .catch((err) => {
            error = "Failed to sign in.";
            my.alert({
              content: "Login Failed: " + err.message,
            });
          })
          .finally(() => {
            isLoading = false;
          });
      },
      fail: (err) => {
        isLoading = false;
        error = "Could not access auth code.";
        my.alert({
          content: "Auth Failed: " + JSON.stringify(err),
        });
      },
    });
  }

  function pay() {
    if (!user || !token) {
      my.alert({ content: "Please login first" });
      return;
    }

    isPaying = true;

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        Authorization: token,
      },
    })
      .then((res) => {
        if (!res.ok) throw new Error("Payment API error");
        return res.json();
      })
      .then((data) => {
        if (!data.paymentUrl) throw new Error("No payment URL");

        my.tradePay({
          paymentUrl: data.paymentUrl,
          success: (res) => {
            my.alert({
              content: "Payment Success: " + JSON.stringify(res),
            });
          },
          fail: (res) => {
            my.alert({
              content: "Payment Failed: " + JSON.stringify(res),
            });
          },
          complete: () => {
            isPaying = false;
          },
        });
      })
      .catch((err) => {
        isPaying = false;
        my.alert({
          content: "Payment Error: " + err.message,
        });
      });
  }
</script>

<main>
  <div class="auth-card animate-enter">

    <!-- SUCCESS STATE -->
    {#if user}
      <div class="auth-header">
        <div style="display:flex;justify-content:center;margin-bottom:1.5rem;">
          <svg width="64" height="64" viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="12" r="12" fill="#dcfce7" />
            <path
              d="M7 13L10 16L17 9"
              stroke="#16a34a"
              stroke-width="2.5"
              stroke-linecap="round"
              stroke-linejoin="round"
            />
          </svg>
        </div>

        <h1 class="auth-title">Success!</h1>
        <p class="auth-subtitle">You are now logged in</p>

        <div
          style="margin-top:1.5rem;padding:1rem;background:var(--background);
          border-radius:12px;font-size:0.9rem;color:var(--text-secondary);"
        >
          ID: {user.id}
        </div>
      </div>
    {/if}

    <!-- ACTIONS -->
    <div class="auth-actions">
      {#if error}
        <div
          style="color:#ef4444;font-size:0.9rem;margin-bottom:0.5rem;
          background:#fee2e2;padding:0.5rem;border-radius:8px;"
        >
          {error}
        </div>
      {/if}

      {#if !user}
        <button
          class="btn btn-primary"
          type="button"
          on:click={loginWithSuperQi}
          disabled={isLoading}
        >
          {isLoading ? "Signing In..." : "Continue with SuperQi"}
        </button>
      {/if}

      {#if user}
        <button
          class="btn btn-secondary"
          type="button"
          on:click={pay}
          disabled={isPaying}
        >
          {isPaying ? "Processing Payment..." : "Payment"}
        </button>
      {/if}
    </div>

  </div>
</main>
