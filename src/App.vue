<template>
  <div>
    <div class="view login" v-if="showLoginForm">
      <form class="login-form" @submit.prevent="login">
        <div class="form-inner">
          <h1>Login to VueChat</h1>
          <label for="username">Username</label>
          <input
            ref="loginUsername"
            type="text"
            v-model="loginUsername"
            placeholder="Please enter your username..."
          />
          <input type="submit" value="Login" />
        </div>
      </form>
    </div>
    <div class="view chat" v-else>
      <header>
        <button class="logout" @click="logout">Logout</button>
        <h1>Welcome, {{ username }}</h1>
      </header>
      <section class="chat-box">
        <div
          v-for="(message, index) in messages"
          :key="index"
          :class="
            message.username == username ? 'message current-user' : 'message'
          "
        >
          <div class="message-inner">
            <div class="username">{{ message.username }}</div>
            <div class="content">{{ message.content }}</div>
          </div>
        </div>
      </section>
      <footer>
        <form @submit.prevent="sendMessage">
          <input
            ref="textMessage"
            type="text"
            v-model="inputMessage"
            placeholder="Write a message..."
          />
          <input type="submit" value="Send" />
        </form>
      </footer>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loginUsername: "",
      inputMessage: "",
      username: "",
      messages: [],
    };
  },
  mounted() {
    // scroll to the bottom of the page when user enters the tab
    window.addEventListener("focus", () => setTimeout(this.scrollToBottom, 0));

    this.polling();
    this.$refs.loginUsername.focus();
  },
  computed: {
    showLoginForm() {
      return this.username === "" || this.username === null;
    },
  },
  methods: {
    login() {
      if (this.loginUsername !== "" || this.loginUsername !== null) {
        this.username = this.loginUsername;

        // reset the login usernam

        this.loginUsername = "";

        setTimeout(() => {
          this.scrollToBottom();
        }, 0);
      }
    },
    logout() {
      this.username = "";
    },
    scrollToBottom() {
      if (document.getElementsByClassName("view chat")[0]) {
        document.getElementsByClassName("view chat")[0].scrollIntoView(false);
      }
    },
    getMessage() {
      const localStorageMessages = this.getItemsFromLocalStorage();

      // add the message in the current state
      this.messages = localStorageMessages;
    },
    sendMessage() {
      // if no message was entered, do nothing
      if (this.inputMessage === "" || this.inputMessage === null) {
        return;
      }

      const localStorageMessages = this.getItemsFromLocalStorage();

      const message = {
        username: this.username,
        content: this.inputMessage,
      };

      localStorageMessages.push(message);
      this.inputMessage = "";

      // TODO: Replace localStorage with an actual POST API call from back-end
      localStorage.setItem("messages", JSON.stringify(localStorageMessages));

      this.getMessage();

      // wait for the message to be shown
      // and afterwards move the scroll to the bottom
      setTimeout(() => {
        this.scrollToBottom();
      }, 0);
    },
    getItemsFromLocalStorage() {
      // check if there are any messages in local storage
      if (localStorage.getItem("messages")) {
        // get the messages from local storage
        return JSON.parse(localStorage.getItem("messages"));
      }
      return [];
    },
    polling() {
      this.getMessage();

      // focus the input message so a click on the input
      // is no longer necessary
      if (this.$refs.textMessage) this.$refs.textMessage.focus();

      // get the data from the API every 500ms
      setTimeout(this.polling, 500);
    },
  },
};
</script>

<style lang="scss">
* {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
.view {
  display: flex;
  justify-content: center;
  min-height: 100vh;
  background-color: #1e6a44;

  &.login {
    align-items: center;
    .login-form {
      display: block;
      width: 100%;
      padding: 15px;

      .form-inner {
        display: block;
        background-color: #fff;
        padding: 50px 15px;
        border-radius: 16px;
        box-shadow: 0px 6px 12px rgba(0, 0, 0, 0.2);
        h1 {
          color: #aaa;
          font-size: 28px;
          margin-bottom: 30px;
        }
        label {
          display: block;
          margin-bottom: 5px;
          color: #aaa;
          font-size: 16px;
          transition: 0.4s;
        }
        input[type="text"] {
          appearance: none;
          border: none;
          outline: none;
          background: none;
          display: block;
          width: 100%;
          padding: 10px 15px;
          border-radius: 8px;
          margin-bottom: 15px;

          color: #333;
          font-size: 18px;
          box-shadow: 0px 0px 0px rgba(0, 0, 0, 0);
          background-color: #f3f3f3;
          transition: 0.4s;
          &::placeholder {
            color: #888;
            transition: 0.4s;
          }
        }
        input[type="submit"] {
          appearance: none;
          border: none;
          outline: none;
          background: none;
          display: block;
          width: 100%;
          padding: 10px 15px;
          background-color: #1e6a44;
          border-radius: 8px;
          color: #fff;
          font-size: 18px;
          font-weight: 700;
          cursor: pointer;
        }
        &:focus-within {
          label {
            color: #1e6a44;
          }
          input[type="text"] {
            background-color: #fff;
            box-shadow: 0 0 6px rgba(0, 0, 0, 0.2);
            &::placeholder {
              color: #666;
            }
          }
        }
      }
    }
  }
  &.chat {
    flex-direction: column;
    header {
      position: relative;
      display: block;
      width: 100%;
      padding: 50px 30px 10px;
      .logout {
        position: absolute;
        top: 15px;
        right: 15px;
        appearance: none;
        border: none;
        outline: none;
        background: none;
        cursor: pointer;

        color: #fff;
        font-size: 18px;
        margin-bottom: 10px;
        text-align: right;
      }
      h1 {
        color: #fff;
      }
    }
    .chat-box {
      border-radius: 24px 24px 0px 0px;
      background-color: #fff;
      box-shadow: 0px 0px 12px rgba(100, 100, 100, 0.2);
      flex: 1 1 100%;
      padding: 30px;
      .message {
        display: flex;
        margin-bottom: 15px;

        .message-inner {
          .username {
            color: #888;
            font-size: 16px;
            margin-bottom: 5px;
            padding-left: 15px;
            padding-right: 15px;
          }
          .content {
            display: inline-block;
            padding: 10px 20px;
            background-color: #f3f3f3;
            border-radius: 999px;
            color: #333;
            font-size: 18px;
            line-height: 1.2em;
            text-align: left;
          }
        }
        &.current-user {
          margin-top: 30px;
          justify-content: flex-end;
          text-align: right;
          .message-inner {
            max-width: 75%;
            .content {
              color: #fff;
              font-weight: 600;
              background-color: #1e6a44;
            }
          }
        }
      }
    }
    footer {
      position: sticky;
      bottom: 0px;
      background-color: #fff;
      padding: 30px;
      box-shadow: 0px 0px 12px rgba(100, 100, 100, 0.2);
      form {
        display: flex;
        input[type="text"] {
          flex: 1 1 100%;
          appearance: none;
          border: none;
          outline: none;
          background: none;
          display: block;
          width: 100%;
          padding: 10px 15px;
          border-radius: 8px 0px 0px 8px;

          color: #333;
          font-size: 18px;
          box-shadow: 0px 0px 0px rgba(0, 0, 0, 0);
          background-color: #f3f3f3;
          transition: 0.4s;
          &::placeholder {
            color: #888;
            transition: 0.4s;
          }
        }

        input[type="submit"] {
          appearance: none;
          border: none;
          outline: none;
          background: none;
          display: block;
          padding: 10px 15px;
          border-radius: 0px 8px 8px 0px;
          background-color: #1e6a44;
          color: #fff;
          font-size: 18px;
          font-weight: 700;
        }
      }
    }
  }
}
</style>
