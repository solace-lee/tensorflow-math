<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
  </div>
</template>

<script>
// @ is an alias to /src

export default {
  name: 'Home',
  created () {
    this.init5()
  },
  methods: {
    init () {
      let i = 0;
      function func() {
        const t = 1;
        i += t;
        return i;
      }
      console.log(func()); // 2

      function func() {
        const t = 2;
        i += t;
        return i;
      }

      console.log(func());// 4
    },

    init1 () {
      var t = 'A';
      function func() {
        var t = 'B';
        function f() {
          return t;
        }
        return f()
      }
      console.log(func()); // B
    },

    init2 () {
      var t = 'A';
      function func() {
        var t = 'B';
        function f() {
          return t;
        }
        return f
      }
      console.log(func()()); // B
    },

    init3 () {
      function f0(){
        try {
          console.log(this.a, this.b);
        } catch (error) {
          console.log('f0', error);
        }
      }
      var a = 1;
      let b = 2;
      
      let obj = {
        a: 3,
        b: 4,
        f1: () => {
          console.log(this.a);
        },
        f2: function () {
          try {
            console.log(this.a);
          } catch (error) {
            console.log('f2', error);
          }
        },
        f3() {
          try {
            console.log(this.a);
          } catch (error) {
            console.log('f3', error);
          }
        },
        f4: f0
      };
      let f5 = obj.f1;
      let f6 = obj.f2;
      let f7 = obj.f3;
      let f8 = obj.f4;
      let f9 = obj.f1.bind(obj);
      let f10 = obj.f2.bind(obj);
      let f11 = obj.f1.bind(this);
      let f12 = obj.f2.bind(this);

      f0(); // undefind
      obj.f1(); // undefind
      obj.f2(); // 3
      obj.f3(); // 3
      obj.f4(); // 3 4
      f5(); // 4
      f6(); // undefind
      f7(); // undefind
      f8(); // undefind
      f9(); // undefind
      f10(); // 3
      f11(); // undefind
      f12(); // undefind

      // 浏览器环境下执行结果
      // f0(); // 1 undefind
      // obj.f1(); // 1
      // obj.f2(); // 3
      // obj.f3(); // 3
      // obj.f4(); // 3 4
      // f5(); // 1
      // f6(); // 1
      // f7(); // 1
      // f8(); // 1 undefind
      // f9(); // 1
      // f10(); // 3
      // f11(); // 1
      // f12(); // 1
    },

    init4 () {
      for (let i = 0; i < 3; i++) {
        setTimeout(function () {
          console.log(i); // 0 1 2
        }, 0)
      }
    },

    init5 () {
      console.log(1);
      new Promise(resolve => {
        console.log(2);
        setTimeout(() => {
          console.log(3);
        }, 10)
        resolve()
        console.log(4);
      }).then(() => {
        console.log(5);
      })

      setTimeout(() => {
        console.log(6);
        Promise.resolve().then(() => {
          console.log(7);
        })
        console.log(8);
      })

      Promise.resolve().then(() => {
        console.log(9);
      })

      console.log(10);
      // 1 2 4 10 5 9 6 8 7 3
    }
  }
}
</script>
