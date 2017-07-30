## 一些改进
本库会将pjax加载内容中的js文件移动到文档的`<head>`标签中,并且对于已经存在于head中的js文件会跳过加载，这对于类库类js文件自然是没有什么问题的，但是对于非类库类js就会出错。
可以通过以下方法避免这一行为：
1. 允许转移到head，但强制重新加载： 在`<script>`标签中设置属性`context`值为`forcedload`。例如`<script src="init.js" context="forcedload"></script>`
2. 禁止转移到head：在`<script>`标签中设置属性`context`值为`inline`。例如`<script src="init.js" context="inline"></script>`

## 用法

### `$.fn.pjax`

最简单和更通用的使用方式如下:

``` javascript
$(document).pjax('a', '#pjax-container')
```

它将会在整个页面的`<a>`标签上启用pjax，当点击该链接的时候，`<a>`标签中所指定的网址内容会被加载到`#pjax-container`元素中。

也可以像下面这样在特定的多个元素上启用pjax:

``` javascript
$(document).pjax('[data-pjax] a, a[data-pjax]', '#pjax-container')
```

#### 参数

`$.fn.pjax`函数说明:

``` javascript
$(document).pjax(selector, [container], options)
```

1. `selector` 是被点击的元素，用符合jquery语法的选择器来表示[$.fn.on].
2. `container` 是显示内容的容器。
3. `options` 选项。支持如下选项：

##### pjax 选项

键 | 默认值 | 说明
----|---------|------------
`timeout` | 650 | ajax超时（以毫秒为单位）。如果达到超时时长，将强制完全刷新
`push` | true | 使用 [pushState][] 添加浏览器历史记录
`replace` | false | 仅仅替换URL，不添加浏览器历史记录
`maxCacheLength` | 20 | 允许保留的最大缓存尺寸
`version` | | 一个字符串或函数。返回当前pjax的版本
`scrollTo` | 0 | 在导航之后滚动到的垂直坐标位置。设置为 `false` 时则不更改位置
`type` | `"GET"` | 请求方法，用于$.ajax。可以查看 [$.ajax][] 了解
`dataType` | `"html"` | 响应数据类型，用于$.ajax。可以查看 [$.ajax][] 了解
`container` | | CSS选择器所指定的元素，其内容会被替换为pjax加载的内容
`url` | link.href | 一个字符串或函数。它返回ajax请求的网址
`target` | link | eventually the `relatedTarget` value for [pjax events](#events)
`fragment` | | 用CSS选择器指定ajax响应内容中的某个元素作为最终内容

您可以通过设置`$.pjax.defaults`的值来更改默认全局选项:

``` javascript
$.pjax.defaults.timeout = 1200
```

### `$.pjax.click`

This is a lower level function used by `$.fn.pjax` itself. It allows you to get a little more control over the pjax event handling.

This example uses the current click context to set an ancestor element as the container:

``` javascript
if ($.support.pjax) {
  $(document).on('click', 'a[data-pjax]', function(event) {
    var container = $(this).closest('[data-pjax-container]')
    var containerSelector = '#' + container.id
    $.pjax.click(event, {container: containerSelector})
  })
}
```

**NOTE** Use the explicit `$.support.pjax` guard. We aren't using `$.fn.pjax` so we should avoid binding this event handler unless the browser is actually going to use pjax.

### `$.pjax.submit`

Submits a form via pjax.

``` javascript
$(document).on('submit', 'form[data-pjax]', function(event) {
  $.pjax.submit(event, '#pjax-container')
})
```

### `$.pjax.reload`

Initiates a request for the current URL to the server using pjax mechanism and replaces the container with the response. Does not add a browser history entry.

``` javascript
$.pjax.reload('#pjax-container', options)
```

### `$.pjax`

Manual pjax invocation. Used mainly when you want to start a pjax request in a handler that didn't originate from a click. If you can get access to a click `event`, consider `$.pjax.click(event)` instead.

``` javascript
function applyFilters() {
  var url = urlForFilters()
  $.pjax({url: url, container: '#pjax-container'})
}
```

## Events

All pjax events except `pjax:click` & `pjax:clicked` are fired from the pjax
container element.

<table>
<tr>
  <th>event</th>
  <th>cancel</th>
  <th>arguments</th>
  <th>notes</th>
</tr>
<tr>
  <th colspan=4>event lifecycle upon following a pjaxed link</th>
</tr>
<tr>
  <td><code>pjax:click</code></td>
  <td>✔︎</td>
  <td><code>options</code></td>
  <td>fires from a link that got activated; cancel to prevent pjax</td>
</tr>
<tr>
  <td><code>pjax:beforeSend</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>can set XHR headers</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:send</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:clicked</code></td>
  <td></td>
  <td><code>options</code></td>
  <td>fires after pjax has started from a link that got clicked</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>before replacing HTML with content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:success</code></td>
  <td></td>
  <td><code>data, status, xhr, options</code></td>
  <td>after replacing HTML content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:timeout</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>fires after <code>options.timeout</code>; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:error</code></td>
  <td>✔︎</td>
  <td><code>xhr, textStatus, error, options</code></td>
  <td>on ajax error; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:complete</code></td>
  <td></td>
  <td><code>xhr, textStatus, options</code></td>
  <td>always fires after ajax, regardless of result</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <th colspan=4>event lifecycle on browser Back/Forward navigation</th>
</tr>
<tr>
  <td><code>pjax:popstate</code></td>
  <td></td>
  <td></td>
  <td>event <code>direction</code> property: &quot;back&quot;/&quot;forward&quot;</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>before replacing content</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>right before replacing HTML with content from cache</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>after replacing content</td>
</tr>
</table>

`pjax:send` & `pjax:complete` are a good pair of events to use if you are implementing a
loading indicator. They'll only be triggered if an actual XHR request is made,
not if the content is loaded from cache:

``` javascript
$(document).on('pjax:send', function() {
  $('#loading').show()
})
$(document).on('pjax:complete', function() {
  $('#loading').hide()
})
```

An example of canceling a `pjax:timeout` event would be to disable the fallback
timeout behavior if a spinner is being shown:

``` javascript
$(document).on('pjax:timeout', function(event) {
  // Prevent default timeout redirection behavior
  event.preventDefault()
})
```

## Advanced configuration

### Reinitializing plugins/widget on new page content

The whole point of pjax is that it fetches and inserts new content _without_
refreshing the page. However, other jQuery plugins or libraries that are set to
react on page loaded event (such as `DOMContentLoaded`) will not pick up on
these changes. Therefore, it's usually a good idea to configure these plugins to
reinitialize in the scope of the updated page content. This can be done like so:

``` js
$(document).on('ready pjax:end', function(event) {
  $(event.target).initializeMyPlugin()
})
```

This will make `$.fn.initializeMyPlugin()` be called at the document level on
normal page load, and on the container level after any pjax navigation (either
after clicking on a link or going Back in the browser).

### Response types that force a reload

By default, pjax will force a full reload of the page if it receives one of the
following responses from the server:

* Page content that includes `<html>` when `fragment` selector wasn't explicitly
  configured. Pjax presumes that the server's response hasn't been properly
  configured for pjax. If `fragment` pjax option is given, pjax will extract the
  content based on that selector.

* Page content that is blank. Pjax assumes that the server is unable to deliver
  proper pjax contents.

* HTTP response code that is 4xx or 5xx, indicating some server error.

### Affecting the browser URL

If the server needs to affect the URL which will appear in the browser URL after
pjax navigation (like HTTP redirects work for normal requests), it can set the
`X-PJAX-URL` header:

``` ruby
def index
  request.headers['X-PJAX-URL'] = "http://example.com/hello"
end
```

### Layout Reloading

Layouts can be forced to do a hard reload when assets or html changes.

First set the initial layout version in your header with a custom meta tag.

``` html
<meta http-equiv="x-pjax-version" content="v123">
```

Then from the server side, set the `X-PJAX-Version` header to the same.

``` ruby
if request.headers['X-PJAX']
  response.headers['X-PJAX-Version'] = "v123"
end
```

Deploying a deploy, bumping the version constant to force clients to do a full reload the next request getting the new layout and assets.


[$.fn.on]: http://api.jquery.com/on/
[$.ajax]: http://api.jquery.com/jQuery.ajax/
[pushState]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries
[plugins]: https://gist.github.com/4283721
[turbolinks]: https://github.com/rails/turbolinks
[railscasts]: http://railscasts.com/episodes/294-playing-with-pjax
