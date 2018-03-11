---
layout: post
title: "About UTF-8"
date: 2018-03-11 14:33:22
tags: code forensics
description: Sample post about UTF-8 encoding
---

## Introduction

UTF-8 was designed for backward compatibility with ASCII.

{% highlight ruby linenos %}
{% raw %}
class String
  def to_codepoint
    self.bytes.each_slice(3).map { |s|
      first = (s[0] & 0xf0) >> 4
      return nil unless first == 0b1110
      res = s[0] & 0xff
      res = res << 6 | s[1] & 0x3f
      res = res << 6 | s[2] & 0x3f
    }
  end
end

chars = File.open(path, "r:UTF-8") { |f| f.read.chomp }
chars.to_codepoint.each { |cp|
  p "U+#{cp.to_s(16).upcase}"
}

{% endraw %}
{% endhighlight xml %}

![img]({{ '/assets/images/utf8.png' | relative_url }}){: .center-image }*utf-8 변환표*
![img]({{ '/assets/images/utf8-trend.png' | relative_url }}){: .center-image }*utf-8 사용 추세*
