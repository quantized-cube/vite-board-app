<template>
  <section>
    <div class="alert h6 text-right" @click="doLogin">
      [login:{{ data.user != null ? data.user.displayName : "---" }}]
    </div>
    <h2>{{ title }}</h2>
    <p class="h5">{{ data.message }}</p>
    <div v-if="data.user" class="alert alert-primary">
      <div class="form-group text-left">
        <label class="h5">Message</label>
        <div class="form-row">
          <div class="col-10">
            <input v-model="data.msg" class="form-control" />
          </div>
          <button @click="add" class="btn btn-primary col-2">投稿</button>
        </div>
      </div>
      <h3 class="my-3">Messages</h3>
      <ul
        v-for="item in data.fire_data"
        :key="item.posted"
        class="list-group text-left"
      >
        <li class="list-group-item">
          <div class="h5">{{ item.msg }}</div>
          <div class="small text-right">
            {{ item.user }} ({{ item.posted }})
          </div>
        </li>
      </ul>
    </div>
    <div v-else>
      <div class="alert alert-warning">※現在、ログインされていません。</div>
    </div>
  </section>
</template>

<script>
import { onMounted, reactive } from "vue";
import { initializeApp } from "firebase/app";
import {
  getDatabase,
  ref,
  child,
  get,
  set,
  query,
  orderByChild,
  limitToLast,
} from "firebase/database";
import { getAuth, signInWithPopup, GoogleAuthProvider } from "firebase/auth";

const firebaseConfig = {
  // Your web app's Firebase configuration
};

const app = initializeApp(firebaseConfig);

const provider = new GoogleAuthProvider();
const auth = getAuth(app);
const dbRef = ref(getDatabase());

export default {
  setup(props) {
    const data = reactive({
      title: "Board",
      message: "ミニ伝言板。最新の投稿を表示します。",
      user: null,
      msg: "",
      num_per_page: 10, //☆取り出すデータ数
      fire_data: {},
    });
    // ログイン処理
    const login = () => {
      signInWithPopup(auth, provider).then((result) => {
        data.user = result.user;
        data.message = "ログインしました。";
        get(
          query(
            child(dbRef, "board"),
            orderByChild("posted"),
            limitToLast(data.num_per_page)
          )
        ).then((snapshot) => {
          if (auth.currentUser != null) {
            let arr = [];
            const res = snapshot.val();
            for (let item in res) {
              arr.unshift(res[item]);
            }
            console.log(arr);
            data.fire_data = arr;
          } else {
            data.fire_data = {};
          }
        });
      });
    };
    // ログアウト処理
    const logout = () => {
      auth.signOut();
      data.user = null;
      data.fire_data = {};
      data.message = "ログアウトしました。";
    };
    // ログイン・ログアウト実行
    const doLogin = () => {
      if (auth.currentUser == null) {
        login();
      } else {
        logout();
      }
    };
    // メッセージ追加
    const add = () => {
      if (auth.currentUser == null) {
        alert("ログインしないと投稿できません。");
        return;
      }
      const user = auth.currentUser;
      console.log(user);
      const d = new Date();
      const dstr =
        d.getFullYear() +
        "-" +
        (d.getMonth() + 1) +
        "-" +
        d.getDate() +
        " " +
        d.getHours() +
        ":" +
        d.getMinutes() +
        ":" +
        d.getSeconds();
      const id = d.getTime();
      const obj = {
        msg: data.msg,
        user: user.displayName,
        posted: dstr,
      };
      set(child(dbRef, "board/" + id), obj);
      data.msg = "";
      data.message = "投稿しました。";
    };
    onMounted(() => {
      if (auth.currentUser == null) {
        login();
      }
      console.log(auth.currentUser);
    });
    return { data, login, logout, doLogin, add };
  },
};
</script>
