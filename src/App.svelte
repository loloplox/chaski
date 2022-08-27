<script>

    import axios from "axios";

    let conversations = []
    let messageContent = ""
    let error = {status: false, message: ""}

    async function getConversation() {
        try {
            const {data} = await axios.get("http://192.168.1.25:8080/shipping/conversation/2/1")

            conversations = data
        } catch (e) {
            error = {status: true, message: e.message}
            console.error(e)
        }
    }

    async function getMessage() {
        let idMessages = []

        for (const conversation of conversations) {
            idMessages.push(conversation.message)
        }

        try {
            const {data} = await axios.post("http://192.168.1.25:8080/messages/conversation", idMessages)

            conversations = conversations.map((conversation, index) => ({...conversation, message: data[index].content}))
        } catch (e) {
            error = {status: true, message: e.message}
            console.error(e)
        }

    }

    async function sendMessage() {
        if (messageContent !== '') {
            const responseMessage = await axios.post(
                "http://192.168.1.25:8080/messages",
                {content: messageContent, insertionDate: new Date().toISOString()}
            )

            const responseShipping =  await axios.post(
                "http://192.168.1.25:8080/shipping",
                {sender: 1, message: responseMessage.data.id, receptor: 2,insertionDate: new Date().toISOString()}
            )

            messageContent = ""

            const newConversation = {
                id: responseShipping.data.id,
                sender: responseShipping.data.sender,
                message: responseMessage.data.content,
                receptor: responseShipping.data.receptor,
                insertionDate: responseShipping.data.insertionDate
            }

            conversations = [...conversations, newConversation]
        }
    }


    getConversation().then(() => {getMessage()})
</script>


<div class="container">
    <div>

        {#each conversations as conversation}
            <h2>{conversation.message}</h2>
        {/each}

        {#if error.status}
            <h1>Error: {error.content}</h1>
        {/if}

    </div>
    <div>
        <input bind:value={messageContent} type="text">
        <button on:click={sendMessage}>Send</button>
    </div>
</div>

