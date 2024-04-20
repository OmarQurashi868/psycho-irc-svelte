<script lang="ts">
  let authtoken = localStorage.getItem("authtoken");
  let serverUrl = localStorage.getItem("serverUrl")
  if (!authtoken || !serverUrl) {
    window.location.pathname = "/";
  }


  let ws = new WebSocket(`ws://${serverUrl}`);

  const getTimestamp = (): string => {
    const time = new Date();
    const dateVal = time.getDate();
    const monthVal = time.getMonth() + 1;
    let hours = time.getHours();
    const minutes = time.getMinutes();
    let m = "AM";

    let date = String(dateVal);
    let month = String(monthVal);

    if (dateVal / 10 < 1) {
      date = "0" + date;
    }
    if (monthVal / 10 < 1) {
      month = "0" + month;
    }

    if (hours > 12) {
      hours -= 12;
      m = "PM";
    }

    const timeString = `${date}/${month}  ${hours}:${minutes} ${m}`;

    return timeString;
  };

  interface Message {
    sender: string;
    type: "message" | "alert";
    content: string;
    timestamp: string;
  }

  let messageList: Message[] = [
    {
      sender: "omar",
      type: "message",
      content: "Hello world! yo brooo oadfoafo",
      timestamp: getTimestamp(),
    },
  ];

  const onSend = async (e: SubmitEvent) => {
    e.preventDefault();
  };
</script>

<div class="card w-96 bg-neutral shadow-xl w-4/5">
  <div class="card-body">
    <!-- Chat display -->
    <div class="input input-bordered flex items-start gap-2 min-h-96">
      {#each messageList as message}
        {#if message.type == "message"}
          <div class="chat chat-start">
            <div class="chat-header">
              {message.sender}
              <time class="text-xs opacity-50">{message.timestamp}</time>
            </div>
            <div class="chat-bubble">{message.content}</div>
          </div>
        {:else}
          <div class="chat-footer opacity-50">
            {message.content}
          </div>
        {/if}
      {/each}
    </div>

    <!-- Chat input -->
    <form class="card-actions justify-end" on:submit={onSend}>
      <label class="input input-bordered flex items-center gap-2 grow">
        <input type="text" class="grow" placeholder="Message" />
      </label>
      <button class="btn btn-primary" type="submit">
        <i class="fa-solid fa-paper-plane"></i>
      </button>
    </form>
  </div>
</div>
