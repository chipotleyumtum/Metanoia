<script>
  import Login from './Login.svelte';
  import ChatMessage from './ChatMessage.svelte';
  import { onMount } from 'svelte';
  import { username, user } from './user';
  import debounce from 'lodash.debounce';
  import GUN from 'gun';
  import { getMessages, sendMessage } from './chatService'; // separate chat logic

  const THREE_HOURS_AGO = new Date(+new Date() - 1 * 1000 * 60 * 60 * 3).toISOString(); // use constants
  const db = GUN();

  let newMessage;
  let messages = [];

  let scrollBottom;
  let lastScrollTop;
  let canAutoScroll = true;
  let unreadMessages = false;

  function autoScroll() {
    setTimeout(() => scrollBottom?.scrollIntoView({ behavior: 'auto' }), 50);
    unreadMessages = false;
  }

  function watchScroll(e) {
    canAutoScroll = (e.target.scrollTop || Infinity) > lastScrollTop;
    lastScrollTop = e.target.scrollTop;
  }

  $: debouncedWatchScroll = debounce(watchScroll, 1000);

  onMount(async () => {
    try {
      messages = await getMessages(db, THREE_HOURS_AGO); // use separate function for getting messages
    } catch (error) {
      console.error('Failed to get messages:', error);
    }
  });

  async function handleSendMessage() {
    try {
      await sendMessage(db, newMessage, user); // use separate function for sending messages
      newMessage = '';
      canAutoScroll = true;
      autoScroll();
    } catch (error) {
      console.error('Failed to send message:', error);
    }
  }
</script>

