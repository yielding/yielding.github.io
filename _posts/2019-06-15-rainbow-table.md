---
title: rainbow table
date: 2019-06-15T01:07:07+0900
description: Rainbow table
share: true
comments: true
published: true
categories:
  - programming
tags:
  - forensics
  - programming
---

## Rainbow table
해시는 포렌식에서 무결성을 검증하는 중요한 도구로 사용된다.
해시 함수의 주요 특징은 다음과 같다.

* 입력이 한 비트만 바뀌어도 결과 전체가 완전하게 바뀐다.(눈사태 효과)
* 두 해시 값이 다르다면 원래의 데이 터도 다르다.
* 같은 해시 값을 갖더라도 원래의 입력값이 같다는 것을 보장하지 않는다.(단사함수가 아님)

특히 첫 번째 특징 때문에 해시는 데이터의 무결성을 검증하는 도구로 사용되고, 결과만 보고 입력을 유추하는 것은 불가능해진다.
그런데, 입력의 경우의 수가 유한한 경우 모든 입력에 대해 해시값을 구한 후 유지하고 있으면, 향후 해시값을 가지고 이 값을 만들어낸 원본 데이터를 찾아낼 수 있는데 이런 자료구조를 레인보우 테이블이라고 한다.

안드로이드의 패턴락 및 카카오톡의 패런락은 사용자의 입력을 내부적으로 해시를 통해 저장하는데, 패턴락을 입력할 수 있는 경우의 수가 유한하기 때문에, 모든 경우의 수에 대한 패턴락을 미리 계산해 놓으면 추후 증거물에서 저장된 해시값을 데이터베이스의 키로 사용하여 원래의 패턴을 찾아낼 수 있다.

아래 화면은 카카오톡의 패턴락 입력화면인데 좌측 상단부터 1, 2, 3, ..., 7, 8, 9의 값을 가진다.

![img]({{ "/assets/images/pattern_lock.jpg" | relative_url }})  *표 1. 카카오톡 패턴락*

1에서 다음으로 이동할 수 있는 경우는 { 2, 4, 5 }. 2에서는 { 1, 3, 4, 5, 6 } 이런 식으로 나누어 볼 수 있는데 이것을 정리해 보면 다음과 같은 자료구조로 표현이 가능하다.

>Ruby: ranbow_table.rb
{:.filename}
{% highlight ruby linenos %}

@candidate_hash = {
  0 => [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ],
  1 => [ 2, 4, 5 ],
  2 => [ 1, 3, 4, 5, 6 ],
  3 => [ 2, 5, 6 ],
  4 => [ 1, 2, 5, 7, 8 ],
  5 => [ 1, 2, 3, 4, 5, 6, 7, 8, 9],
  6 => [ 2, 3, 5, 8, 9],
  7 => [ 4, 5, 8 ],
  8 => [ 4, 5, 6, 7, 9 ],
  9 => [ 5, 6, 8 ]
}

{% endhighlight %}

위의 코드에서 키 '0'은 사용자가 패턴을 시작하는 첫 지점, 즉 아홉 개의 점에서 어떤 점에서도 선택을 할 수 있다는 것을 표시하고 나머지 1 ~ 9의 경우는 각 점에서 이동할 수 있는 인접한 점(8-neighbor)을 표시한다.

이 데이터를 기반으로 'backtracking'을 해서 사용자가 입력가능한 모든 패턴의 경우를 나열하고 이 데이터를 기반으로 레인보우 테이블을 구성하여, 실제 증거물에서 발견된 해시를 기반으로 원래 패턴을 찾아낸다. 레인보우 테이블 구성을 위한 'backtracking' 코드는 다음과 같다.

>Ruby: ranbow_table.rb
{% highlight ruby linenos %}
#!/usr/bin/env ruby

require "deep_clone"
require "digest/sha1"

class Passcode
  def initialize
    @sha1 = Digest::SHA1.new
    @passcode = {}
    @candidates_hash = {
      0 => [1, 2, 3, 4, 5, 6, 7, 8, 9], # for start

      1 => [2, 4, 5],
      2 => [1, 3, 4, 5, 6],
      3 => [2, 5, 6],
      4 => [1, 2, 5, 7, 8],
      5 => [1, 2, 3, 4, 6, 7, 8, 9],
      6 => [2, 3, 5, 8, 9],
      7 => [4, 5, 8],
      8 => [4, 5, 6, 7, 9],
      9 => [5, 6, 8]
    }

    prepare_rainbow_table
  end

  def passcode_of hash
    @passcode[hash]
  end

private
  def prepare_rainbow_table
    4.upto(9) { |i| backtrack([], 0, i) }
  end

  def process arr
    @sha1.reset
    arr.each { |e| @sha1 << (e-1).chr }
    s = arr.map(&:to_s).reduce(:+) # [1, 2, 3, 4] => "1234"
    e = @sha1.hexdigest.upcase
    @passcode[e] = s
    printf("%s => %s\n",e, s)
  end

  def backtrack(arr, beg, depth)
    process(arr) && return if arr.length == depth

    @candidates_hash[beg].each do |c|
      unless arr.include?(c)
        backtrack(DeepClone.clone(arr).push(c), c, depth)
      end
    end
  end
end

decryptor = Passcode.new
p decryptor.passcode_of("59A3556203C8F6C908D6C6BCE4C5E03F6BF343E3")

{% endhighlight %}
