# discuz相关学习

***

具体见discuz帮助手册及教学视频

模板文件附录：http://faq.comsenz.com/library/template/filelist/filelist_index.htm

开发文档：http://open.discuz.net/?ac=document&page=dev

论档介绍：http://faq.comsenz.com/library/x3/front/forum/forumindex.html

论坛资料库：http://faq.comsenz.com/library/?from=discuzheader

## discuz首页

全局->站点功能->开启门户

界面->导航设置->设门户为首页。

在这里可以看到那个页面是主页。

门户首页：portal/index.htm或者index.php

论坛首页：forum/discuz.htm或者discuz.php

(为什么用htm而不是html，是因为discuz模板处理过程中为了以php三个字符同个数，方便文件名的截取处理)

## discuz模板解析语法

1. <!--{}-->

   discuz用<!--{}-->来解析语法。这和html注释类似，多了个括号可以方便查看，又可方便html代码编写。

   discuz论坛首页模板为./template/forum/discuz.html

   直接输出变量，等同于<?php echo $my_var;?>



2. {lang forum_category_modedby}

   {lang forum_category_modedby}为语言包使用

   其中语言包在./source/language/目录下，以php数组形式存放

   具体：http://bbs.zb7.com/discuz/dx25/template/syntax/syntax_index.htm

   处理后的模板为html与php混合的文件，即原生态文件，

3. 在表单及css中if..else可以简写

   形如：{if ** }**{/if}

   ```html
   {if widthauto()}{lang switch_narrow}{else}{lang switch_wide}{/if}
   <td class="fl_g"{if $forumcolwidth} width="$forumcolwidth"{/if}>
   ```

   参考：http://faq.comsenz.com/library/template/syntax/syntax_index.htm

## discuz模板后缀

模板处理类为：class/class_template.php

后缀名为：

.htm ：也会执行php程序

.php：会执行里面的php程序

```php
$basefile = basename(DISCUZ_ROOT.$tplfile, '.htm');
		$file == 'common/header' && defined('CURMODULE') && CURMODULE && $file = 'common/header_'.CURMODULE;
		$this->file = $file;

		if($fp = @fopen(DISCUZ_ROOT.$tplfile, 'r')) {
			$template = @fread($fp, filesize(DISCUZ_ROOT.$tplfile));
			fclose($fp);
		} elseif($fp = @fopen($filename = substr(DISCUZ_ROOT.$tplfile, 0, -4).'.php', 'r')) {
			$template = $this->getphptemplate(@fread($fp, filesize($filename)));
			fclose($fp);
		} else {
			$tpl = $tpldir.'/'.$file.'.htm';
			$tplfile = $tplfile != $tpl ? $tpl.', '.$tplfile : $tplfile;
			$this->error('template_notfound', $tplfile);
		}
```

其实discuz只是把他们当作普通文件处理，最终解析为.php。后缀为.htm或者.php都可以，只是对.php文件进行处理时，默认忽略第一行。

```php
//这里仅仅是把第一行去掉了
	function getphptemplate($content) {
		$pos = strpos($content, "\n");
		return $pos !== false ? substr($content, $pos + 1) : $content;
	}
```

## 目录权限

## php面向对象

面向过程是流式执行，面向对象先编译后执行，所以有时用与声明的顺序可以改变。由其是在php中，没有严格约束。