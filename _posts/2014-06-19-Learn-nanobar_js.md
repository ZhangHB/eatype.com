---
layout: posts
title: 尝试着理解 Nanobar.js
---

## {{ page.title }}

[Nanobar.js](https://github.com/jacoborus/nanobar/) 是个原生 js 写成的小插件。用来在页面顶端创建一个进度条，显示页面加载的状态。当然，这个脚本只负责前端显示部分，至于在页面加载的过程中，什么时候显示百分之多少，就不是这个脚本该负责的了。

因为是原生 js 写成的，理解它的运行机制，应该能让我这样的菜鸟学到不少新东西。所以我尝试着给这个脚本添加注释，权当是阅读笔记了。


    /* http://nanobar.micronube.com/  ||
       https://github.com/jacoborus/nanobar/
       MIT LICENSE */

    var Nanobar = (function(){
        'use strict';

        // 定义变量
        // 两个主要对象
        /*  Bar：
                - 构造器函数
                - 这个对象存储了用来展示宽度的 bar 元素 el
                - 和目标宽度的值 here
                - 还有用来标识动画状态的 moving

            Nanobar：
                - 构造器函数
                - 这个对象存储了用来在 DOM 里定位的元素，
                  Bar 的 el 会被绑定到这个元素上
                - 还有用来调用 Bar.go() 的 go() 方法

            init：
                - 这个函数衔接了 Nanobar 和 Bar 这两个对象

            move()：
                - 用来递归修改宽度，产生动画，判断状态，
                  并清理无用的 DOM 元素的方法

        */

        var addCss, Bar, Nanobar, move, place, init,

            // 容器的样式
            cssCont = {
                width: '100%',
                height: '4px',
                zIndex: 9999,
                top : '0'
            },

            // 进度条的样式
            cssBar = {
                width:0,
                height: '100%',
                clear: 'both',
                transition: 'height .3s'
            };





        // add `css` to `el` element
        // 被调用时，接收俩参数，
        // 第一个参数是添加样式的元素，第二个参数应该是对象
        // 遍历第二个参数的所有属性，
        // 当成css绑定到第一个参数传进来的元素上
        addCss = function (el, css ) {
            var i;
            for (i in css) {
                el.style[i] = css[i];
            }
            el.style.float = 'left';
        };





        // animation loop
        // 动画
        // 这个函数在 Bar.prototype.go 里被调用执行
        // 并且会自行进行递归以产生动画
        move = function () {

            // 话说，这里的 this 应该是指调用这个函数的上下文环境
            // 看来看去，也就是在下边的 Bar.prototype.go 里调用了


            // 这个 this 指的是 Bar 对象
            // 这个 Bar 对象，保存了各种必要的属性
            // 比如：
            // 实时计算出来的宽度；
            // 目标宽度
            // 要处理的 bar 元素
            // bar 元素的颜色
            // 还有将会调用本函数的 go 方法
            var self = this,
                // 这里把 this 赋值给 self，
                // 是为了在下边的 setTimeout 里，
                // 通过 self 变量访问 this

                // 计算要运动的距离
                // this.here 是目标宽度，也就是将要变成的宽度
                // this.width 是目前的宽度
                // 这俩值都是百分比宽度，
                // 并且会在 place 方法里指定到 bar 上
                dist = this.width - this.here;

            // （#1）如果目标宽度和当前宽度的差不到百分之 0.01
            if (dist < 0.1 && dist > -0.1) {

                // call 是函数的全局方法，
                // 作用是在使用一个指定的 this 值和若干个指定的
                // 参数值的前提下调用某个函数或方法.
                // place 是自定义方法，用来直接修改 bar 的宽度
                // 这里是因为，目标宽度和当前宽度的差不到百分之
                // 0.01，说明已经无限接近目标宽度了，
                // 干脆直接把目标宽度赋值给 bar
                place.call( this, this.here );

                // 修改标识，标示一下，动画已经停止了
                this.moving = false;

                // 如果 Bar 的 width 属性变成 100 了，
                // 也就是说，宽度到头儿了
                if (this.width == 100) {

                    // 就把 用来显示宽度的 div 元素的高度清零
                    this.el.style.height = 0;

                    // 下边这些代码确保了，
                    // 在 bar 宽度到达 100% 之后的 0.3 秒时，
                    // 把刚才处理的 bar 元素干掉，
                    // 保证当前页面不会出现太对的 DOM 节点
                    setTimeout( function () {
                        self.cont.el.removeChild( self.el );
                    }, 300);
                }
            } else {
                // 如果（#1）的判断不成立，
                // 说明当前的宽度还没到达目标宽度
                // 那就继续每隔16毫秒，计算并设置一次 Bar 的宽度，形成动画
                // 计算的方式是每隔16毫秒计算一下当前宽度和目标宽度的距离差的4分之一
                // 赋值给 Bar 的 width 属性
                place.call( this, this.width - (dist/4) );

                setTimeout( function () {
                    // 这里的 this 指向了 window 对象，所以在函数开始的地方，把正确的 this 赋值给了 self 变量
                    // 然后递归调用 this 也就是 Bar 对象的 go 方法
                    self.go();
                }, 16);
            }
        };





        // set bar width
        // 设置 bar 的宽
        place = function (num) {

            // 这个 this.width 只是修改 Bar 对象的 width 属性，这个属性是用来判断状态的
            // 不是用来修改 显示宽度动画的元素
            this.width = num;

            // 这里才是修改显示宽度动画的实际元素的宽度
            this.el.style.width = this.width + '%';
        };





        // create and insert bar in DOM and this.bars array
        // 创建并把 bar 插到 DOM 里，同时插入到 Nanobar.bars 属性头部，这个属性是个数组
        // 这个函数在 Nanobar 被实例化时调用，所以这里的 this 指的是 Nanobar 对象
        init = function () {

            // 创建了一个 Bar 的实例
            // init 这个方法在 Nanobar 里被调用，所以把 this 作为参数传递给了 Bar()，
            // Bar() 里边的 cont 就被直接指向了 Nanobar 构造器所创建出来的那个实例
            var bar = new Bar( this );

            // 这句也应该能说明，this 是指向 Nanobar 的，
            this.bars.unshift( bar );

            // 比如可以这样显示 Nanobar 的各种属性和方法
            console.log(this.el);
            // console.log(this.go);
            // console.log(this.opts);
            // console.log(this.bars);
        };




        // 在 init() 里被声明成一个实例
        Bar = function ( cont ) {

            // create progress element
            // 创建 进度条 元素

            // 创建一个空 div
            this.el = document.createElement( 'div' );

            // 把默认的背景色设置到这个 div 上
            this.el.style.backgroundColor = cont.opts.bg;

            // 为了更好的理解代码，我在这里给刚才创建的空 div 添加了一个 class
            // 结果显示，这个空 div 是内部的那个用来显示宽度的 div
            this.el.className = "this-el";

            // 把 Bar 对象的 width 属性 清零
            this.width = 0;

            // 把 Bar 对象的目标宽度清零
            this.here = 0;

            // 把用来标示是否在进行动画的标志设成false，表示未在进行动画
            this.moving = false;

            // 把 Bar 的 cont 属性设置为参数传进来的对象
            this.cont = cont;

            // 给刚才创建的空 div 设置样式
            addCss( this.el, cssBar);

            // 为了更好的理解代码，我在这里给所谓的 cont.el 添加了一个 class
            // 结果显示，这个 cont.el 是外部的容器 div
            cont.el.className = "cont-el";

            // 把 this.el 绑定到 cont.el 上
            // cont.el 是在
            cont.el.appendChild( this.el );
        };




        // 这个方法被 Nanobar.go 调用
        // 传进来的参数，决定了 bar 的目标宽度
        Bar.prototype.go = function (num) {

            // 判断一下是否传进来了参数，如果有
            if (num) {

                // 把传进来的参数，赋值给目标宽度
                this.here = num;

                // 看一眼 动画标识，是否是动画中，如果不是
                if (!this.moving) {

                    // 标识为 动画中，标识 bar 正在不停地改变宽度
                    this.moving = true;

                    // 调用运动函数，真正改变 bar 的宽度
                    move.call( this );
                }

                // 如果动画标识已经是 动画中 了，
            } else if (this.moving) {

                // 调用运动函数，真正改变 bar 的宽度
                move.call( this );
            }
        };




        // 这个构造器在被实例化的时候，创建了相应的dom元素，并设置了相应的样式且被放在了dom的正确位置上，
        // 然后调用了 init 函数
        Nanobar = function (opt) {

            var opts = this.opts = opt || {},
                el;

            // set options
            opts.bg = opts.bg || '#000';
            this.bars = [];


            // create bar container
            // 创建 bar 的容器
            el = this.el = document.createElement( 'div' );

            // 给刚才创建出来的 el 增加一个自定义属性，以利于在分析这些代码的时候，在 DOM 里查看到它
            // 不过话说，不加这个也能明白，就是外部的 div
            el.setAttribute("myAttrib", "Nanobar-el");

            // append style
            // 添加样式，并按照参数设定 id
            addCss( this.el, cssCont);
            if (opts.id) {
                el.id = opts.id;
            }
            // set CSS position
            // 设置定位方法，如果设置了参数 target，说明需要把 bar 的外部容器放在 target 元素里
            // 如果没设置参数 target，就说明将会把 bar 放在 body 里，
            // 相应的，给两种不同的方式，添加不同的样式
            el.style.position = !opts.target ? 'fixed' : 'relative';

            // insert container
            // 插入到相应的 DOM 里
            if (!opts.target) {
                document.getElementsByTagName( 'body' )[0].appendChild( el );
            } else {
                opts.target.insertBefore( el, opts.target.firstChild);
            }


            // 当 Nanobar 被实例化的时候，init 函数就被执行了
            // 这里调用 init 是为了把 Nanobar 作为参数，传递到 Bar() 里
            init.call( this );
        };





        Nanobar.prototype.go = function (p) {
            // expand bar
            // 调用 Bar.go 这个方法，改变 bar 宽度
            this.bars[0].go( p );

            // 这行是我添加的，就是想把，每次调用 init 方法时，unshift 到 bars 数组，
            // 并且每次当目标宽度为 100% 时调用 init 方法的实际情况变成可视化的
            // console.log(this.bars.length);

            // create new bar at progress end
            // 当 bar 的宽度到达 100% 之后，调用一下 init 方法，重新创建一个 bar 元素
            if (p == 100) {
                init.call( this );
            }
        };

        return Nanobar;
    })();


------

### 尝试描述一下程序的流程

1. 这个脚本里处理的 DOM 对象都是在脚本里创建的；
2. 在 html 页面，执行 var nanobar = new Nanobar() 时，就创建了一个 Nanobar 的实例；
3. 创建这个实例的同时，Nanobar 函数就被运行了，相应的属性和元素就被创建出来，并被设置了相应的样式且被放到了 DOM 里相应的位置上，之后 init 函数被调用；
4. init 被调用的时候，把它所在的上下文环境传递给了 Bar 构造器，并创建了一个 Bar 的实例，随后这个实例被 unshift 到 Nanobar 的 bars 属性里；
5. 实例化 Bar 的过程中，创建了内部的 div，并绑定到 外部 div 里；
6. 在 html 页面，执行 nanobar.go(50) 的时候，会调用 Bar 实例的 go 方法，并且调用的是 bars 属性里的第一个 Bar 实例；
7. 也就是说 nanobar(50) 等于 Bar.go(50);
8. Bar.go() 方法会调用 move() 函数，判断并动态改变内部 div 的宽度，形成动画；
9. move() 函数调用 place() 函数，进行具体的 DOM 操作