
---

title: Trie 树

date: 2019-01-24 12:45:16 +0800

tags: []

---
2013年09月23日 02:14:27 [arhaiyun](https://me.csdn.net/arhaiyun) 阅读数：9467

**Trie 树， **又称字典树，单词查找树。它来源于retrieval(检索)中取中间四个字符构成(读音同try)。用于存储大量的字符串以便支持快速模式匹配。主要应用在信息检索领域。

Trie 有三种结构： 标准trie (standard trie)、压缩trie、[后缀trie(suffix trie)](http://hxraid.iteye.com/blog/620414) 。 最后一种将在[《字符串处理4：后缀树》](https://www.baidu.com/s?wd=%E3%80%8A%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%A4%84%E7%90%864%EF%BC%9A%E5%90%8E%E7%BC%80%E6%A0%91%E3%80%8B&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)中详细讲，这里只将前两种。

**1. 标准Trie (standard trie)**

**标准 Trie树的结构 **： 所有含有公共前缀的字符串将挂在树中同一个结点下。实际上trie简明的存储了存在于串集合中的所有公共前缀。 假如有这样一个字符串集合X{bear,bell,bid,bull,buy,sell,stock,stop}。它的标准Trie树如下图：

![](http://hxraid.iteye.com/upload/picture/pic/57141/51209a0f-3903-36fd-b3b2-19afa62c8cf2.jpg#width=)

上图（蓝色圆形结点为内部结点，红色方形结点为外部结点），我们可以很清楚的看到字符串集合X构造的Trie树结构。其中从根结点到红色方框叶子节点所经历的所有字符组成的串就是字符串集合X中的一个串。

注意这里有一个问题： 如果X集合中有一个串是另一个串的前缀呢？ 比如，X集合中加入串bi。那么上图的Trie树在绿色箭头所指的内部结点i 就应该也标记成红色方形结点。这样话，一棵树的枝干上将出现两个连续的叶子结点(这是不合常理的)。

也就是说字符串集合X中不存在一个串是另外一个串的前缀 。如何满足这个要求呢？我们可以在X中的每个串后面加入一个特殊字符$(这个字符将不会出现在字母表中)。这样，集合X{bear$、bell$、.... bi$、bid$}一定会满足这个要求。

总结：一个存储长度为n，来自大小为d的字母表中s个串的集合X的标准trie具有性质如下：

(1) 树中每个内部结点至多有d个子结点。

(2) 树有s个外部结点。

(3) 树的高度等于X中最长串的长度。

(4) 树中的结点数为O(n)。

**标准 Trie树的查找**

对于英文单词的查找，我们完全可以在内部结点中建立26个元素组成的指针数组。如果要查找a，只需要在内部节点的指针数组中找第0个指针即可(b=第1个指针，随机定位)。时间复杂度为O(1)。

查找过程：假如我们要在上面那棵Trie中查找字符串bull (b-u-l-l)。

(1) 在root结点中查找第('b'-'a'=1)号孩子指针，发现该指针不为空，则定位到第1号孩子结点处——b结点。

(2) 在b结点中查找第('u'-'a'=20)号孩子指针，发现该指针不为空，则定位到第20号孩子结点处——u结点。

(3) ... 一直查找到叶子结点出现特殊字符'$'位置，表示找到了bull字符串

如果在查找过程中终止于内部结点，则表示没有找到待查找字符串。

效率：对于有n个英文字母的串来说，在内部结点中定位指针所需要花费O(d)时间，d为字母表的大小，英文为26。由于在上面的算法中内部结点指针定位使用了数组随机存储方式，因此时间复杂度降为了O(1)。但是如果是中文字，下面在实际应用中会提到。因此我们在这里还是用O(d)。 查找成功的时候恰好走了一条从根结点到叶子结点的路径。因此时间复杂度为O(d*n)。

但是，当查找集合X中所有字符串两两都不共享前缀时，trie中出现最坏情况。除根之外，所有内部结点都自由一个子结点。此时的查找时间复杂度蜕化为O(d*(n^2))

**标准 Trie树的Java代码实现：**
```java
/*
StandarTire.java
Trie 树， 又称字典树，单词查找树。
它来源于retrieval(检索)中取中间四个字符构成(读音同try)。用于存储大量的字符串以便支持快速模式匹配。主要应用在信息检索领域。
@author arhaiyun
date:2013/09/23
*/
 
import java.util.*;
 
enum NodeKind{LN, BN};
 
/**
*Trie node
*/
class TrieNode
{
	char key;
	TrieNode[] points = null;
	NodeKind kind = null;
}
 
/**
* Branch node
*/
class BranchNode extends TrieNode
{
	BranchNode(char k)
	{
		super.key = k;
		super.kind = NodeKind.BN;
		super.points = new TrieNode[27];
	}
}
 
 
/**
* Leaf node
*/
class LeafNode extends TrieNode
{
	LeafNode(char k)
	{
		super.key = k;
		super.kind = NodeKind.LN;
	}
}
 
 
public class StandardTrie
{
	//Create root node
	TrieNode root = new BranchNode(' ');
 
	//[1].Insert a word into tire tree
	public void insert(String words)
	{
		TrieNode curNode = root;
		//add '$' as an end symbol
		words = words + "$";
		char[] chars = words.toCharArray();
		
		for(int i = 0; i < chars.length; i++)
		{
			if(chars[i] == '$')
			{
				curNode.points[26] = new LeafNode('$');
			}
			else
			{
				int pSize = chars[i] - 'a';
				// If not exists creat a new branch node
				if(curNode.points[pSize] == null)
				{
					curNode.points[pSize] = new BranchNode(chars[i]);
				}
				curNode = curNode.points[pSize];
			}
		}
	}
	
	//[2].Check if a word is in tire tree
	public boolean fullMatch(String words)
	{
		TrieNode curNode = root;
		char[] chars = words.toCharArray();
		
		for(int i = 0; i < chars.length; i++)
		{
			int pSize = chars[i] - 'a';
			System.out.print(chars[i]+"->");
			if(curNode.points[pSize] == null)
				return false;
			curNode = curNode.points[pSize];
		}
		
		if(curNode.points[26] != null && curNode.points[26].key == '$')
			return true;
		
		return false;	
	}
	
	
	//[3].preorder root traverse
	private void preorderTraverse(TrieNode curNode)
	{
		if(curNode != null)
		{
			System.out.print(curNode.key);
			
			if(curNode.kind == NodeKind.BN)
			{
				for(TrieNode node : curNode.points)
				{
					preorderTraverse(node);
				}
			}
			else
				System.out.println();
		}
	}
	
	//[4].Get root node
	public TrieNode getRoot()
	{
		return this.root;
	}
	
	
	public static void main(String[] args)
	{
		StandardTrie trie = new StandardTrie();
		
		trie.insert("amazon");
		trie.insert("yahoo");
		trie.insert("haiyun");
		trie.insert("baidu");
		trie.insert("alibaba");
		trie.insert("offer");
		trie.insert("stock");
		trie.insert("stop");
		
		trie.preorderTraverse(trie.getRoot());
		
		System.out.println(trie.fullMatch("yahoo"));
		System.out.println(trie.fullMatch("yaho"));
		System.out.println(trie.fullMatch("baidu"));
		System.out.println(trie.fullMatch("alibaba"));
	}
}
```


