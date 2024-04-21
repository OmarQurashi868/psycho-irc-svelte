<script lang="ts">
  let authtoken = localStorage.getItem("authtoken");
  let serverUrl = localStorage.getItem("serverUrl");
  if (!authtoken || !serverUrl) {
    window.location.pathname = "/";
  }

  let msgInput = "";

  let ws = new WebSocket(`ws://${serverUrl}`);

  ws.addEventListener("error", (event) => {
    console.log(event);
    window.location.pathname = "/";
  });

  ws.addEventListener("message", (event) => {
    const msg = JSON.parse(event.data)
    
    const newMsg: Message = {
      sender: msg["sender"],
      type: msg["type"],
      content: msg["content"],
      timestamp: getTimestamp(),
      isMine: false,
    };

    messageList = [...messageList, newMsg];
  });

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
    isMine: boolean;
  }

  let messageList: Message[] = [
    {
      sender: "omar",
      type: "message",
      content: "Hello world! yo brooo oadfoafo",
      timestamp: getTimestamp(),
      isMine: false,
    },
  ];

  const onSend = async (e: SubmitEvent) => {
    e.preventDefault();
    if (msgInput == "") return;

    const newMsg: Message = {
      sender: "You",
      type: "message",
      content: msgInput,
      timestamp: getTimestamp(),
      isMine: true,
    };

    messageList = [...messageList, newMsg];

    ws.send(msgInput);

    msgInput = "";
  };

  const onDisconnect = () => {
    document.location.pathname = "/";
  };
</script>

<div class="card bg-neutral shadow-xl w-5/6 md:w-4/5">
  <div class="card-body">
    <div class="flex align-center justify-between">
      <button class="btn btn-secondary self-start" on:click={onDisconnect}
        ><i class="fa-solid fa-plug-circle-xmark"></i>
      </button>
      <span class="self-center">SERVER NAME</span>
    </div>
    <!-- Chat display -->
    <div
      class="input input-bordered flex flex-col items-start gap-2 min-h-96 overflow-auto"
    >
      {#each messageList as message}
        {#if message.type == "message"}
          <div
            class="chat"
            class:chat-start={!message.isMine}
            class:chat-end={message.isMine}
            class:self-end={message.isMine}
          >
            <div class="chat-header">
              {!message.isMine ? message.sender : ""}
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
        <input
          type="text"
          class="grow"
          placeholder="Message"
          bind:value={msgInput}
        />
      </label>
      <button class="btn btn-primary" type="submit">
        <i class="fa-solid fa-paper-plane"></i>
      </button>
    </form>
  </div>
</div>
