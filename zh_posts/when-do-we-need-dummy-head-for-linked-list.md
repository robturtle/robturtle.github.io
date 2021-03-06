---
title: 我们什么时候需要给 linked list 加上 dummy head
date: 2016-07-15 01:51:51
tags:
- ADT
- DS

---

> 今天地铁上被问到这个问题，考虑到已经是连续不同三个人问了这个问题，我就在此处一并作答并做一些引申。

先放最后的结论，而关于这个问题的回答，可以用一句话概括：

> We will need a dummy head whenever we need a **functional** empty list

这个问题之所以会出现，完全是因为对数据结构设计不够完备，而语言本身又不提供相应表达能力所导致的。为什么这么说呢，因为在我们这里讨论的对链表的实现中，当链表为空的时候和当链表不为空的时候，它们所支持的能力并不一致。

<!-- more -->

如果我们把链表作为一种抽象数据类型 (ADT: Abstract Data Type) 来看待的话，它应该支持这些能力：

- 值节点能够访问/修改自身
- 访问/修改到下一节点的引用

而在我们这里讨论的  Java 下的链表实现中，我们仅用一个 `ListNode` 类表示非空链表，那么唯一合乎情理的表示空链表的形式就只剩下 `null` 了。可是，很显然，`null` 并不支持本该支持的第二项能力。我们也可以说，我们的这一链表实现，并不是一个完备的链表 ADT 。当加上了 `dummy head` 后 ，我们才让这个数据结构表现统一了。

举个例子，删除节点的算法可以描述成如下：

```haskell
delete x from A->x->B = A->B
```

其中，`->` 代表连接两个链表，`x` 为我们欲删除的节点。那么，显然算法的结果就应当是将 `x` 前的链表和 `x` 后的链表连接起来即可。当 `x` 为第一个节点时，它前面的链表即为空链表，而在我们简陋的实现里用来表示空链表的 `null` 由于缺乏“修改下一节点的引用”的能力，使得“重新连接“这一动作无法被实现。

可以说，并不存在判断何时需要 dummy head 的问题，我们所有的链表实现都应当具备这一结构让空链表依然满足能力约定，使得它成为一个完备的链表 ADT 实现。只是因为在某些算法里我们并没有碰到违反约定的情况，从而给人一种不需要 dummy 的观感而已。

那么，回到问题，什么时候需要 dummy 呢？当我们需要用到一个满足能力约定的空链表之时咯。

**引申2**：

上文里我说到这个问题还可以归咎于语言设计，这又是何解呢？就是因为 Java 里 `null` 这一坏的设计，让几乎所有使用 `null` 的设计都变成了对 `null` 的滥用。在大部分的情况下，“空”的概念依然具备某些能力，这就需要我们显式地定义出来。对于链表来说，就应当是类似这样：

```java
interface ListNode<T> { ListNode<T> getNext(); }
class ValueNode<T> implements ListNode<T> {
  T getValue() { return this.value; }
  ListNode<T> getNext() { return this.next; }
  ...
}
class EmptyNode implements ListNode<?> {
  ListNode<?> getNext() { return this.next; }
  ...
}
```

设 `e` 为空链表，由于下面三个不变式：

1. `e->A = A` 
2. `A->e = A`
3. `A->e->B = A->B`

我们可以设计 `e->X->e` 为所有链表的 regular form 来保证所有操作的合法性。当然，我们也可以引用一个代理类：

```java
List<T> { final ListNode<T> dummy = new ListNode<>(); ... }
```

让该类来保证链表始终处于 regular form 的状态。当然有的时候我们也可以使用 irregular form 的形式，因为允许任意插入空链表会使得很多算法的实现变得简单优雅许多。而这，恰恰是链表 ADT 原本就赋予它的能力。