<script>
  // @ts-nocheck

  let isLoading = false;
  let isPaying = false;
  let user = null;
  let error = null;

  function loginWithSuperQi() {
    isLoading = true;
    error = null;

    // Call SuperQi/Hylid Bridge to get Auth Code
    // @ts-ignore
    my.getAuthCode({
      scopes: ["auth_base", "USER_ID"],
      success: (res) => {
        // Exchange Auth Code for Token via API
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
            if (!response.ok) throw new Error("Network response was not ok");
            return response.json();
          })
          .then((data) => {
            console.log("Login success:", data);
            user = data.record;
            // Optional: Store token if needed
            // localStorage.setItem('token', data.token);
          })
          .catch((err) => {
            console.error("Login API Error:", err);
            error = "Failed to sign in. Please try again.";
            // @ts-ignore
            my.alert({
              content: "Login Failed: " + err.message,
            });
          })
          .finally(() => {
            isLoading = false;
          });
      },
      fail: (res) => {
        console.error("Auth Code Error:", res);
        isLoading = false;
        error = "Could not access auth code.";
        // @ts-ignore
        my.alert({
          content: "Auth Failed: " + JSON.stringify(res),
        });
      },
    });
  }

  function pay() {
    isPaying = true;
    // @ts-ignore - SuperQi/Hylid payment bridge
    fetch('https://its.mouamle.space/api/payment', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': token
                },
            }).then(res => res.json()).then(data => {
                if (data.paymentUrl) {
                    // @ts-ignore
                    my.tradePay({
                        paymentUrl: data.paymentUrl,
                        success: (res) => {
                            // @ts-ignore
                            my.alert({
                                content: 'Payment Success: ' + JSON.stringify(res),
                            });
                        },
                        fail: (res) => {
                            // @ts-ignore
                            my.alert({
                                content: 'Payment Failed: ' + JSON.stringify(res),
                            });
                        },
                        complete: () => {
                            isPaying = false;
                        },
                    });
                } else {
                    throw new Error('No payment URL returned');
                }
            }).catch(err => {
                console.error('Payment API Error:', err);
                // @ts-ignore
                my.alert({
                    content: 'Payment Failed: ' + err.message,
                });
                isPaying = false;
            });
  }
 
</script>

<main>
  <div class="auth-card animate-enter">
    {#if user}
      <!-- Success State -->
      <div class="auth-header">
        <div
          style="display: flex; justify-content: center; margin-bottom: 1.5rem;"
        >
          <svg
            width="64"
            height="64"
            viewBox="0 0 24 24"
            fill="none"
            xmlns="http://www.w3.org/2000/svg"
          >
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
          style="margin-top: 1.5rem; padding: 1rem; background: var(--background); border-radius: 12px; font-size: 0.9rem; color: var(--text-secondary);"
        >
          ID: {user.id}
        </div>
      </div>
    {:else}
      <!-- Login State -->
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
        {#if error}
          <div
            style="color: #ef4444; font-size: 0.9rem; margin-bottom: 0.5rem; background: #fee2e2; padding: 0.5rem; border-radius: 8px;"
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
      <button
        style="margin-top: 1rem;"
        class="btn btn-secondary"
        type="button"
        on:click={handlePayment}
        disabled={isLoading || isPaying}
      >
        {#if isPaying}
          Processing Payment...
        {:else}
          Payment
        {/if}
      </button>
    </div>
</main>
