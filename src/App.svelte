<script>
  // @ts-nocheck

  let isLoading = false;
  let isPaying = false;

  let authCode = "";
  let token = "";
  let userId = null;
  let error = null;

  /* ======================
     AUTHENTICATE (LOGIN)
     ====================== */
  function authenticate() {
    isLoading = true;
    error = null;

    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        authCode = res.authCode;

        fetch("https://its.mouamle.space/api/auth-with-superQi", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            token: authCode, // 🔴 نفس كود المدرّب بالضبط
          }),
        })
          .then((res) => res.json())
          .then((data) => {
            token = data.token;
            userId = data?.record?.id || null;

            my.alert({
              content: "Login successful",
            });
          })
          .catch((err) => {
            error = "Login failed";
            my.alert({
              content: "Error: " + JSON.stringify(err),
            });
          })
          .finally(() => {
            isLoading = false;
          });
      },
      fail: (res) => {
        isLoading = false;
        my.alert({
          content: "Auth failed",
        });
      },
    });
  }

  /* ======================
     PAYMENT
     ====================== */
  function pay() {
    if (!token) {
      my.alert({
        content: "Please login first",
      });
      return;
    }

    isPaying = true;

    fetch("https://its.mouamle.space/api/payment", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": token, // 🔴 نفس المدرّب
      },
    })
      .then((res) => res.json())
      .then((data) => {
        my.tradePay({
          paymentUrl: data.url, // 🔴 نفس المدرّب
          success: () => {
            my.alert({
              content: "Payment successful",
            });
          },
        });
      })
      .catch(() => {
        my.alert({
          content: "Payment failed",
        });
      })
      .finally(() => {
        isPaying = false;
      });
  }
</script>

<main>
  <div class="auth-card animate-enter">
    <div class="auth-header">
      <h1 class="auth-title">Demo MiniApp</h1>
      <p class="auth-subtitle">SuperQi Mini App</p>
    </div>

    <div class="auth-actions">
      <button
        class="btn btn-primary"
        on:click={authenticate}
        disabled={isLoading}
      >
        {#if isLoading}
          Signing in...
        {:else}
          Login
        {/if}
      </button>

      <button
        class="btn btn-secondary"
        on:click={pay}
        disabled={isPaying}
      >
        {#if isPaying}
          Processing...
        {:else}
          Pay
        {/if}
      </button>

      {#if authCode}
        <p class="text-sm">Auth Code: {authCode}</p>
      {/if}

      {#if userId}
        <p class="text-sm">User ID: {userId}</p>
      {/if}

      {#if error}
        <p class="text-sm text-red-500">{error}</p>
      {/if}
    </div>
  </div>
</main>
