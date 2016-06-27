+++
title = "Giới thiệu các tính năng mới trong Docker 1.12"
date = "2016-06-26T16:56:43+02:00"
tags = ["Docker", "DockerCon2016", "DevOps"]
categories = ["DevOps"]
menu = ""
images = ["static/images/dockerswarm01.png"]
banner = "banners/dockercon2016.png"
+++

DockerCon đã giới thiệu một loạt các tính năng mới trong Docker, và với version 1.12 (hiện tại đang rc2) chúng ta đã có thể bắt đầu vọc phá. Hôm nay tôi xin giới thiệu về [Docker Swarm Mode](https://docs.docker.com/engine/swarm/), một tính năng mới có trong Docker Engine (v1.12).

Nếu bạn đang sử dụng các phiên bản trước v1.12.0-rc1, vui lòng xem thêm [ở đây](https://docs.docker.com/swarm/).


## Docker Swarm Mode là gì?
Trong phiên bản v1.12.0, Docker Swarm là một tính năng được tích hợp sẵn trong Docker Engine, và chúng ta có thể xây dựng một swarm cluster, tạo các service trong cluster một cách dễ dàng mà không phải cài thêm bất kỳ phần mềm nào. Đặc biệt, version này cũng bao gồm việc xử lý về vấn đề Security, Networking, State & Cluster Initialization.

Xem thêm ở đây để được giới thiệu về [Docker Swarm Mode](https://www.youtube.com/watch?v=KC4Ad1DS8xU)

## Cài đặt
Trong bài này, chúng ta sẽ tạo ra một swarm cluster gồm 1 cluster manager và 3 worker. Nếu như trước đây chúng ta cần thêm các container khác cho service discovery, load balancer, ... để xây dựng một cluster thì bây giờ mấy thứ đó không cần nữa. Tất cả đã được built-in trong Docker Engine. 

![Environment Setup]
(/images/dockerswarm01.png)

### Mở port để giao tiếp giữa các hosts
- **TCP port 2377**: port này để cluster mananegement
- **TCP** và **UDP port 7946** để giao tiếp giữa các nodes
- **TCP** và **UDP port 4789** dành cho overlay network

Nếu chúng ta sử dụng Boot2Docker phiên bản mới nhất thì nó đã làm sẵn cho chúng ta rồi, nice :)

### Dùng docker-machine để tạo các máy ảo
Lưu ý là chúng ta phải cài Docker Toolbox để có thể xài lệnh docker-machine nhé. Trước đây tôi cũng đã từng nhầm lẫn giữa Docker for Mac & Docker Toolbox, lưu ý đây là 2 sản phẩm hoàn toàn khác nhau nhé.

Chúng ta tạo ra các máy ảo như sau:

* **swarm-00**: máy này sẽ làm cluster manager
* **swarm-01**: worker số 1, IP Address sẽ là: 192.168.99.100
* **swarm-02**: worker số 2
* **swarm-03**: worker số 3

```
$ docker-machine create -d virtualbox swarm-00
$ docker-machine create -d virtualbox swarm-01
$ docker-machine create -d virtualbox swarm-02
$ docker-machine create -d virtualbox swarm-03
```

I assume you've Git installed. Inside the folder of your Hugo site run

    $ mkdir themes
    $ cd themes
    $ git clone git@github.com:digitalcraftsman/hugo-icarus-theme.git

You should see a folder called `hugo-icarus-theme` inside the `themes` directory that we created a few moments ago. For more information read the official [setup guide](https://gohugo.io/overview/installing/) of Hugo.


## Setup

In the next step navigate to the `exampleSite` folder at `themes/hugo-icarus-theme/exampleSite/`. Its structure should look similar to this:

    exampleSite
    ├── config.toml
    ├── content
    │   └── post
    │       ├── creating-a-new-theme.md
    │       ├── go-is-for-lovers.md
    │       ├── hugo-is-for-lovers.md
    │       ├── introducing-icarus-and-its-features.md
    │       ├── linked-post.md
    │       └── migrate-from-jekyll.md
    ├── data
    │   └── l10n.toml
    └── static
        └── banners
            └── placeholder.png

In order to get your site running, you need to copy `config.toml` and `data/l10n.toml` into the root folders.


## The config file

Now, let us take a look into the `config.toml`. Feel free to play around with the settings.


### Comments

The optional comment system is powered by Disqus. Enter your shortname to enable the comment section under your posts.

    disqusShortname = ""

Tip: you can disable the comment section for a single page in its frontmatter:

```toml
+++
disable_comments = true
+++
```


### Menu

You can also define the items menu entries as you like. First, let us link a post that you've written. We can do this in the frontmatter of the post's content file by setting `menu` to `main`. Take a look at the menu if you want to see a live example.

    +++
    menu = "main"
    +++

Furthermore, we can add entries that don't link to posts. Back in the `config.toml` you'll find a section for the menus:

    [[params.menu]]
        before = true
        label  = "Home"
        link   = "/"

Define a label and enter the URL to resource you want to link. With `before` you can decide whether the link should appear before **or** after all linked posts in the menu. `Home` appears before the linked post, `Tags` and `Categories` after them (as in the menu above).


### Sidebars

In order to use the full width of the website you can disable the profile on the left and / or the widgets on the right for a single page in the frontmatter:

```toml
+++
disable_profile = true
disable_widgets = true
+++
```


### Tell me who you are

Maybe you noticed the profile section on the left. Add your social network accounts to the profile section on the left by entering your username under `social`. The links to your account will be create automatically.


### Widgets

On the right, you can see some useful widgets that you can activate as you like. You can deactivate them under `params.widgets`:

    [params.widgets]
        recent_articles = false
        categories = true
        tags = true
        tag_cloud = true


## Localization (l10n)

You don't blog in English and you want to translate the theme into your native locale? No problem. Take a look in the `data` folder and you'll find a file `l10n.toml` that we've copied at the beginning. It contains all strings related to the theme. Just replace the original strings with your own.


## Linking thumbnails

After creating a new post you can define a banner by entering the relative path to the image.

    banner = "banners/placeholder.png"

This way you can store them either next to the content file or in the `static` folder.


## Mathematical equations

Mathematical equations in form of LaTeX or MathML code can be rendered with the support of [MathJax](https://www.mathjax.org). MathML works out of the box. If you're using LaTeX you need to wrap your equation with `$$` as shown in the following example:

    $$ z = r \cdot (\sin{\phi} + \cos{\phi} \cdot i) $$

$$ z = r \cdot (\sin{\phi} + \cos{\phi} \cdot i) $$

You can also print formulas inline: $a^2 + b^2 = c^2$. In this case wrap the formula only once with `$`.


## Shortcodes

Last but not least I included some useful [shortcodes](http://gohugo.io/extras/shortcodes/) to make your life easier.

### Gallery

This way you can include a gallery into your post. Copy the code below into your content file and enter the relative paths to your images.

    {{</* gallery
        "/banners/placeholder.png"
        "/banners/placeholder.png"
        "/banners/placeholder.png"
    */>}}

<p></p>

{{< gallery "/banners/placeholder.png" "/banners/placeholder.png" "/banners/placeholder.png" >}}


### JSFiddle

It works the same with JSFiddle examples you want to showcase. The parameter `id` consists of the username and id of the example.

    {{</* jsfiddle id="zalun/NmudS" */>}}

<p></p>

{{< jsfiddle id="zalun/NmudS" >}}

As descibed in the [docs of JSFiddle](http://doc.jsfiddle.net/use/embedding.html), you can define which tabs will be shown. Enter the tabs you want to see separated by a comma in the `tabs` parameter.

    {{</* jsfiddle id="zalun/NmudS" tabs="html,result" */>}}

Do you see the difference?

{{< jsfiddle id="zalun/NmudS" tabs="html,result" >}}


## Nearly finished

In order to see your site in action, run Hugo's built-in local server.

    $ hugo server

Now enter [`localhost:1313`](http://localhost:1313) in the address bar of your browser.


## Contributing

Have you found a bug or got an idea for a new feature? Feel free to use the [issue tracker](//github.com/digitalcraftsman/hugo-icarus-theme/issues) to let me know. Or make directly a [pull request](//github.com/digitalcraftsman/hugo-icarus-theme/pulls).


## License

This theme is released under the MIT license. For more information read the [License](https://github.com/digitalcraftsman/hugo-icarus-theme/blob/master/LICENSE.md).


## Annotations

Thanks to [Steve Francia](//github.com/spf13) for creating Hugo and the awesome community around the project.
