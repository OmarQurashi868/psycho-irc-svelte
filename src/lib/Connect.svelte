<script lang="ts">
    let registerMode = false;

    let server = localStorage.getItem("server") || "";
    let username = localStorage.getItem("username") || "";
    let password = "";
    let passwordConfirmation = "";

    let error = "";

    let isLoading = false;

    const onRegisterModeChange = () => {
        passwordConfirmation = "";
    };

    const onErrorClear = () => {
        error = "";
    };

    const redirectToChat = () => {
        window.location.pathname = "/chat"
    }

    const onSubmit = async (e: SubmitEvent) => {
        e.preventDefault();

        isLoading = true;
        error = "";

        if (registerMode && password != passwordConfirmation) {
            error = "Passwords do not match";
            isLoading = false;
            return;
        }

        if (
            username == "" ||
            password == "" ||
            (registerMode && passwordConfirmation == "")
        ) {
            error = "Missing credentials";
            isLoading = false;
            return;
        }

        if (server == "") {
            error = "Missing server address";
            isLoading = false;
            return;
        }

        
        let serverUrl = server;
        
        if (
            !serverUrl.startsWith("http://") ||
            !serverUrl.startsWith("https://")
        ) {
            serverUrl = "http://" + serverUrl;
        }

        if (serverUrl.includes("localhost")) {
            serverUrl.replace("localhost", "127.0.0.1");
        }
        
        localStorage.setItem("serverUrl", serverUrl.replace("http://", ""));
        localStorage.setItem("username", username);

        let statusCode = 0;

        // Register
        if (registerMode) {
            const body = {
                username,
                password,
                passwordConfirmation,
            };

            try {
                fetch(`${serverUrl}/users/register`, {
                    method: "POST",
                    body: JSON.stringify(body),
                    headers: {
                        "Content-type": "application/json",
                    },
                })
                    .then((response) => {
                        statusCode = response.status;
                        return response.json();
                    })
                    .then((data) => {
                        if (statusCode != 200) {
                            error = data["message"];
                            isLoading = false;
                            return;
                        }
                        localStorage.setItem("authtoken", data["token"]);
                        redirectToChat();
                    });
            } catch (err: any) {
                error = err;
                isLoading = false;
            }
        } else {
            // Login
            const authHeader = {
                "Content-Type": "text/plain",
                Authorization: "Basic " + btoa(`${username}:${password}`),
            };
            try {
                fetch(`${serverUrl}/users/login`, {
                    method: "GET",
                    headers: authHeader,
                })
                    .then((response) => {
                        statusCode = response.status;
                        return response.json();
                    })
                    .then((data) => {
                        if (statusCode != 200) {
                            error = data["message"];
                            isLoading = false;
                            return;
                        }
                        localStorage.setItem("authtoken", data["token"]);
                        redirectToChat();
                    });
            } catch (err: any) {
                error = err;
                isLoading = false;
            }
        }
    };
</script>

{#if error}
    <div class="toast toast-top toast-center">
        <div role="alert" class="alert alert-error flex flex-row">
            <i class="fa-solid fa-xmark"></i>
            <span>{error}</span>
        </div>
    </div>
{/if}
<form class="card w-96 bg-neutral shadow-xl" on:submit={onSubmit}>
    <!-- Server Address -->
    <div class="card-body">
        <label class="input input-bordered flex items-center gap-2">
            <i class="fa-solid fa-server"></i>
            <input
                type="text"
                class="grow"
                placeholder="IP:Port"
                bind:value={server}
                on:change={onErrorClear}
                disabled={isLoading}
            />
        </label>

        <div class="divider" />

        <!-- Login mode check -->
        <div class="flex gap-2">
            <div class:badge-outline={registerMode} class="badge badge-accent">
                Login
            </div>
            <input
                type="checkbox"
                class="toggle toggle-sm"
                bind:checked={registerMode}
                on:change={onRegisterModeChange}
                disabled={isLoading}
            />
            <div class:badge-outline={!registerMode} class="badge badge-accent">
                Register
            </div>
        </div>

        <!-- Username -->
        <label class="input input-bordered flex items-center gap-2">
            <i class="fa-solid fa-user"></i>
            <input
                type="text"
                class="grow"
                placeholder="Username"
                bind:value={username}
                on:change={onErrorClear}
                disabled={isLoading}
            />
        </label>

        <!-- Password -->
        <label class="input input-bordered flex items-center gap-2">
            <i class="fa-solid fa-key"></i>
            <input
                type="password"
                class="grow"
                placeholder="Password"
                bind:value={password}
                on:change={onErrorClear}
                disabled={isLoading}
            />
        </label>

        <!-- Password Confirmation -->
        <label
            class:hidden={!registerMode}
            class="input input-bordered flex items-center gap-2"
        >
            <i class="fa-solid fa-key"></i>
            <input
                type="password"
                class="grow"
                placeholder="Password Confirmation"
                bind:value={passwordConfirmation}
                on:change={onErrorClear}
                disabled={isLoading}
            />
        </label>

        <!-- Submit -->
        <div class="card-actions justify-end">
            <button
                disabled={isLoading}
                class="btn btn-primary"
                type="submit"
            >
                {#if isLoading}
                    <span class="loading loading-dots loading-md text-primary"
                    ></span>
                {:else}
                    Connect
                {/if}
            </button>
        </div>
    </div>
</form>
