<template>
  <div class="dive">
    <div class="path">{{ "/" + path.join("/") }}</div>

    <div class="back" @dblclick="backtrack()" title="上一層" tabindex="0">
      <img src="../assets/path.png" />...
    </div>

    <div class="files">
      <img src="../assets/load.gif" v-show="load" class="load" />
      <div
        class="file"
        v-for="path in files"
        :key="path.id"
        @dblclick="next(path)"
        :title="path.name"
        tabindex="0"
      >
        <span class="info"
          ><img src="../assets/path.png" v-if="path.isDirectory" /> {{ path.name
          }}<span v-if="!path.isDirectory">{{ path.size }}</span></span
        >
      </div>
    </div>

    <div class="preview" v-show="filepr">
      <span class="menu"
        ><a :href="url" :download="file"><img src="../assets/download.png" class="x" /></a
        ><img src="../assets/x.svg" @click="filepr = false" class="x"
      /></span>
      <div class="filepreview">
        <img :src="url" class="img" v-show="filetype === 'img'" />
        <div v-show="filetype === 'code'" class="code">
          <pre class="prettyprint" id="www">{{ code }}</pre>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: "Dive",
  props: {
    admin: Boolean,
  },
  data() {
    return {
      mainpath: [],
      path: [],
      files: [],
      filepr: false,
      load: false,
      filetype: "",
      url: "",
      file: "",
      code: "",
    };
  },
  methods: {
    next: function (path1) {
      if (path1.isDirectory) {
        this.path.push(path1.name);
        console.log(this.path);
        this.files = [];
        this.load = true;
        fetch("/api/path/?path=" + encodeURIComponent(path1.path), {
          method: "GET",
          headers: { "Content-Type": "application/json" },
        })
          .then((req) => {
            return req.json();
          })
          .then((data) => {
            this.load = false;
            this.files = data;
          });
      } else {
        if (
          path1.name.search(/.+\.((jpg)|(png)|(gif))|(GIF)|(JPG)|(PNG)|(JPEG)|(jpeg)/) !==
          -1
        ) {
          this.filepr = true;
          this.file = path1.name;
          this.filetype = "img";
          this.url = `/api/path/?path=${encodeURIComponent(path1.path)}`;
        } else if (
          path1.name.search(/.+\.((py)|(js)|(cs))|(css)|(html)|(kt)|(cpp)|(c)/) !== -1
        ) {
          this.filetype = "code";
          this.code = "";
          this.filepr = true;
          fetch("/api/path/?path=" + encodeURIComponent(path1.path), {
            method: "GET",
            headers: { "Content-Type": "application/json" },
          })
            .then((req) => {
              return req.text();
            })
            .then((text) => {
              this.code = text;
              document.getElementById("www").className = "prettyprint";
            });
        }
      }
    },
    backtrack: function () {
      this.path.splice(this.path.length - 1, 1);
      this.files = [];
      this.load = true;
      fetch("/api/path/?path=" + encodeURIComponent(this.path.join("\\")), {
        method: "GET",
        headers: { "Content-Type": "application/json" },
      })
        .then((req) => {
          return req.json();
        })
        .then((data) => {
          this.load = false;
          this.files = data;
        });
    },
  },
  beforeMount() {
    fetch("/api/path/", {
      method: "GET",
      headers: { "Content-Type": "application/json" },
    })
      .then((req) => {
        return req.json();
      })
      .then((req) => {
        this.mainpath = req;
        this.files = req;
        console.log(req);
      });
  },
};
</script>
<style scoped>
.file {
  width: 80%;
  height: 48px;
  cursor: pointer;
  padding-top: 24px;
  border-bottom: 0px;
  border-top: 0.5px;
  border-left: 0px;
  border-right: 0px;
  border-block-color: rgba(39, 38, 38, 0.534);
  border-style: inset;
  user-select: none;
  background-color: rgba(0, 0, 0, 0);
}
.file:hover,
.file:focus {
  background-color: rgba(70, 66, 66, 0.562);
  outline: none;
}
.preview {
  position: fixed;
  height: 100%;
  width: 100%;
  top: 0px;
  left: 0px;
  background-color: rgba(31, 30, 30, 0.733);
}
.filepreview {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.back {
  cursor: pointer;
  user-select: none;
  margin-top: 20px;
}
.back:hover,
.back:focus {
  background-color: rgba(70, 66, 66, 0.562);
  outline: none;
}
.load {
  margin-left: 50%;
  margin-top: 20%;
  width: 48px;
  height: 48px;
  user-select: none;
}
.x {
  width: 36px;
  height: 36px;
  cursor: pointer;
  user-select: none;
  margin-left: 13px;
}
.menu {
  position: fixed;
  right: 0;
}
.img {
  user-select: none;
  max-height: 100%;
}
.code {
  overflow-y: scroll;
  max-height: 100%;
  background-color: rgb(180, 180, 180);
}
.pln {
  color: rgb(255, 255, 255);
}
</style>
