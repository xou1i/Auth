<script>
  // @ts-nocheck

  let isLoading = false;
  let isPaying = false;
  let user = null;
  let error = null;
  let token = null;

  /* ======================
     LOGIN WITH SUPERQI
     ====================== */
  function loginWithSuperQi() {
    isLoading = true;
    error = null;

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
            if (!response.ok) {
              throw new Error("Login API error");
            }
            return response.json();
          })
          .then((data) => {
            user = data.record;
            token = data.token; // مهم للدفع
            console.log("LOGIN SUCCESS:", data);
          })
          .catch((err) => {
            console.error(err);
            error = "Failed to sign in";
            my.alert({
              content: "Login Failed",
            });
          })
          .finally(() => {
            isLoading = false;
          });
      },
      fail: (err) => {
        console.error(err);
        isLoading = false;
        error = "Auth failed";
        my.alert({
          content: "Auth Failed",
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
        "Authorization": token, // نفس API مال المدرّب
      },
    })
      .then((res) => {
        if (!res.ok) {
          throw new Error("Payment API error");
        }
        return res.json();
      })
      .then((data) => {
        console.log("PAYMENT RESPONSE:", data);

        if (!data.url && !data.paymentUrl) {
          throw new Error("No payment URL returned");
        }

        my.tradePay({
          paymentUrl: data.paymentUrl || data.url,
          success: (res) => {
            my.alert({
              content: "Payment successful",
            });
          },
          fail: (res) => {
            my.alert({
              content: "Payment failed",
            });
          },
          complete: () => {
            isPaying = false;
          },
        });
      })
      .catch((err) => {
        console.error(err);
        my.alert({
          content: err.message || "Payment failed",
        });
        isPaying = false;
      });
  }
</script>

<main>
  <div class="auth-card animate-enter">
    {#if user}
      <!-- SUCCESS STATE -->
      <div class="auth-header">
        <div style="display:flex;justify-content:center;margin-bottom:1.5rem;">
          <svg width="64" height="64" viewBox="0 0 24 24">
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
          style="margin-top:1.5rem;padding:1rem;background:var(--background);border-radius:12px;font-size:0.9rem;"
        >
          ID: {user.id}
        </div>
      </div>

      <div class="auth-actions">
        <button
          class="btn btn-secondary"
          type="button"
          on:click={pay}
          disabled={isPaying}
        >
          {#if isPaying}
            Processing Payment...
          {:else}
            Payment
          {/if}
        </button>
      </div>
    {:else}
      <!-- LOGIN STATE -->
      <div class="auth-header">
        <svg
          width="48"
          height="48"
          viewBox="0 0 24 24"
          style="margin-bottom:1rem;"
        >
          <rect width="24" height="24" rx="12" fill="var(--primary-light)" />
          <circle cx="12" cy="12" r="3" fill="var(--primary)" />
        </svg>

        <h1 class="auth-title">Welcome Back</h1>
        <p class="auth-subtitle">Sign in to continue</p>
      </div>

      <div class="auth-actions">
        {#if error}
          <div
            style="color:#ef4444;font-size:0.9rem;margin-bottom:0.5rem;"
          >
            {error}
          </div>
        {/if}

        <button
          class="btn btn-primary"
          type="button"
          on:click={loginWithSuperQi}
          disabled={isLoading}
        >
          {#if isLoading}
            Signing In...
          {:else}
            Continue with SuperQi
          {/if}
        </button>
      </div>
    {/if}
  </div>
</main>
