<script lang="ts">
    let registerMode = false;

    let server = "";
    let username = "";
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

    const onSubmit = async () => {
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

        // Register
        if (registerMode) {
            const body = {
                username,
                password,
                passwordConfirmation
            };

            fetch(`${server}/users/register`, {
                method: "POST",
                body: JSON.stringify(body),
                headers: {
                    "Content-type": "application/json",
                },
            })
                .then((response) => response.json())
                .then((data) => (error = data));
        }

        // Login
        const authHeader = {
            "Content-Type": "text/plain",
            "Authorization": "Basic " + btoa(`${username}:${password}`),
        };

        fetch(`${server}/users/login`, {
            method: "GET",
            headers: authHeader,
        })
            .then((response) => response.json())
            .then((json) => (error = json));
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
<div class="card w-96 bg-neutral shadow-xl">
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
                on:click={onSubmit}
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
</div>
