<script>
  import Pusher from 'pusher-js';

  const pusher = new Pusher('a7fe0a77fb100e0402d7', {
    cluster: 'ap2',
    encrypted: true,
    authEndpoint: 'https://the-quiz-task.herokuapp.com/pusher/auth'
  });

  export default ({
    getPresenceID () {
      // This function checks the address bar of the browser for params
      let getQueryString = (field, url) => {
        let href = url ? url : window.location.href;
        let reg = new RegExp('[?&]' + field + '=([^&#]*)', 'i');
        let string = reg.exec(href);
        return string ? string[1] : null;
      }
      // Appends 'presence' to the result
      let id = getQueryString('id');
      id = 'presence-' + id;
      return id;
    },
    subscribeToPusher (presenceid) {
      if(presenceid) {
        presenceid = 'presence-' + presenceid;
      } else {
        presenceid = this.getPresenceID();
      }
      console.log(presenceid);
      let channel = pusher.subscribe(presenceid);
      return channel;
    }
  })
</script>