

## 算法

### 常见的查找算法

#### 线性查找

执行搜索的最常见的方法是将每个项目与我们正在寻找的数据进行比较，这就是线性搜索或顺序搜索。它是执行搜索的最基本的方式。如果列表中有n项。在最坏的情况下。我们必须搜索n个项目才能找到一个特定的项目。下面遍历一个数组来查找一个项目。

```php
function linearSearch(array $arr, int $needle) {
    for ($i = 0, $count = count($arr); $i < $count; $i++) {
        if ($needle === $arr[$i]) {
            return true;
        }
    }

    return false;
}
```

线性查找的复杂度

![img](https://pic4.zhimg.com/80/v2-55183ff95f23ee7b630630ff8c14652b_1440w.jpg)



#### 二分搜索

线性搜索的平均时间复杂度或最坏时间复杂度是O(n)，这不会随着待搜索数组的顺序改变而改变。所以如果数组中的项按特定顺序排序，我们不必进行线性搜索。我们可以通过执行选择性搜索而可以获得更好的结果。最流行也是最著名的搜索算法是“二分搜索”。虽然有点像二叉搜索树，但我们不用构造二叉搜索树就可以使用这个算法。

```php
function binarySearch(array $arr, int $needle) {
  $low = 0;
  $high = count($arr) - 1;
  
  while ($low <= $high) {
    $middle = (int)(($low + $high) / 2);
    if ($arr[$middle] < $needle) {
      $low = $middle + 1;
    } else if ($arr[$middle] > $needle) {
      $high = $middle - 1;
    } else {
      return true;
    }
  }
  return false;
}
```

在二分搜索算法中，我们从数据的中间开始，检查中间的项是否比我们要寻找的项小或大，并决定走哪条路。这样，我们把列表分成两半，一半完全丢弃，像下面的图像一样。

![img](https://pic4.zhimg.com/80/v2-78e0e75e0874f00533612e1b5164269f_1440w.jpg)

递归版本：

```php
function binarySearchRecursion(array $arr, int $needle, int $low, int $high) {
  if ($high < $low) return false;
  $middle = (int)(($low + $high) / 2);
  if ($arr[$middle] < $needle) {
    return binarySearchRecursion($arr, $needle, $middle + 1, $high);
  } else if ($arr[$middle] > $needle) {
    return binarySearchRecursion($arr, $needle, $low, $middle - 1);
  } else {
    return true;
  }
}
```

二分搜索复杂度分析

对于每一次迭代，我们将数据划分为两半，丢弃一半，另一半用于搜索。在分别进行了1，2次和3次迭代之后，我们的列表长度逐渐减少到n/2，n/4，n/8...。因此，我们可以发现，k次迭代后，将只会留下n/2^k项。最后的结果就是 n/2^k = 1，然后我们两边分别取对数 得到 k = log(n)，这就是二分搜索算法的最坏运行时间复杂度。

![img](https://pic2.zhimg.com/80/v2-6d5a5373d8a2dc29b935a2bfb0686a1d_1440w.jpg)

#### 重复二分查找

有这样一个场景，假如我们有一个含有重复数据的数组，如果我们想从数组中找到2的第一次出现的位置，使用之前的算法将会返回第5个元素。然而，从下面的图像中我们可以清楚地看到，正确的结果告诉我们它不是第5个元素，而是第2个元素。因此，上述二分搜索算法需要进行修改，将它修改成一个重复的搜索，搜索直到元素第一次出现的位置才停止。

![img](https://pic1.zhimg.com/80/v2-db40d8091811d3372d995c98ca66352c_1440w.jpg)

```php
function repetitiveBinarySearch(array $data, int $needle){
  $low = 0;
  $high = count($data) - 1;
  $firstIndex = -1;
  while ($low <= $high) {
    $middle = ($low + $high) >> 1;
    if ($data[$middle] == $needle) {
      $firstIndex = $middle;
      $high = $middle - 1;
    } else if ($data[$middle] > $needle) {
      $high = $middle - 1;
    } else {
      $low = $middle + 1;
    }
  }
  return $firstIndex;
}
```

**小思考**

对于一个包含n个项目的数组，并且它们没有排序。由于我们知道二分搜索更快，我们决定先对其进行排序，然后使用二分搜索。但是，我们清楚最好的[排序算法](https://link.zhihu.com/?target=https%3A//segmentfault.com/a/1190000016325416)，其最差的时间复杂度是O(nlogn)，而对于二分搜索，最坏情况复杂度是O（logn)。所以，如果我们排序后应用二分搜索，复杂度将是O（nlogn）。

但是，我们也知道，对于任何线性或顺序搜索（排序或未排序），最差的时间复杂度是O（n），显然好于上述方案。

考虑另一种情况，即我们需要多次搜索给定数组。我们将k表示为我们想要搜索数组的次数。如果k为1，那么我们可以很容易地应用之前的线性搜索方法。如果k的值比数组的大小更小，暂且使用n表示数组的大小。如果k的值更接近或大于n，那么我们在应用线性方法时会遇到一些问题。假设k = n，线性搜索将具有O（n2）的复杂度。现在，如果我们进行排序然后再进行搜索，那么即使k更大，一次排序也只会花费O（nlogn）时间复。然后，每次搜索的复杂度是O（logn），n次搜索的复杂度是O（nlogn）。如果我们在这里采取最坏的运行情况，排序后然后搜索k次总的的复杂度是O（nlogn），显然这比顺序搜索更好。

我们可以得出结论，如果一些搜索操作的次数比数组的长度小，最好不要对数组进行排序，直接执行顺序搜索即可。但是，如果搜索操作的次数与数组的大小相比更大，那么最好先对数组进行排序，然后使用二分搜索。

二分搜索算法有很多不同的版本。我们不是每次都选择中间索引，我们可以通过计算作出决策来选择接下来要使用的索引。我们现在来看二分搜索算法的两种变形：插值搜索和指数搜索。





```php
function interpolationSearch(array $arr, int $needle) {
    $low = 0;
    $high = count($arr) - 1;
    while ($arr[$low] != $arr[$high] && $needle >= $arr[$low] && $needle < $arr[$high]) {
        $middle = intval($low + ( ($needle - $arr[$low]) * ($high - $low) / ($arr[$high] - $arr[$low]) ));
        
        if ($arr[$middle] < $needle) {
            $high = $middle - 1;
        } else if ($arr[$middle] > $needle) {
            $low = $middle + 1;
        } else {
            return $middle;
        }
    }
    if ($needle == $arr[$low]) {
        return $low;
    }
    return -1;
}
```

插值搜索需要更多的计算步骤，但是如果数据是均匀分布的，这个算法的平均复杂度是O(log(log n))，这比二分搜索的复杂度O(logn)要好得多。 此外，如果值的分布不均匀，我们必须要小心。 在这种情况下，插值搜索的性能可以需要重新评估。下面我们将探索另一种称为指数搜索的二分搜索变体。

#### 插值搜索

在二分搜索算法中，总是从数组的中间开始搜索过程。 如果一个数组是均匀分布的，并且我们正在寻找的数据可能接近数组的末尾，那么从中间搜索可能不是一个好选择。 在这种情况下，插值搜索可能非常有用。插值搜索是对二分搜索算法的改进，插值搜索可以基于搜索的值选择到达不同的位置。例如，如果我们正在搜索靠近数组开头的值，它将直接定位到到数组的第一部分而不是中间。使用公式计算位置，如下所示

![img](algorithm.assets/v2-12264ce182c300fbcfa115e617bb186a_720w.png)

可以发现，我们将从通用的mid =（low * high)/2 转变为更复杂的等式。如果搜索的值更接近arr[high]，则此公式将返回更高的索引，如果值更接近arr[low]，则此公式将返回更低的索引.

```php
function interpolationSearch(array $arr, int $needle) {
    $low = 0;
    $high = count($arr) - 1;

    while ($arr[$low] != $arr[$high] && $needle >= $arr[$low] && $needle <= $arr[$high]) {
        $middle = intval($low + ($needle - $arr[$low]) * ($high - $low) / ($arr[$high] - $arr[$low]));

        if ($arr[$middle] < $needle) {
            $low = $middle + 1;
        } elseif ($arr[$middle] > $needle) {
            $high = $middle - 1;
        } else {
            return $middle;
        }
    }

    if ($needle == $arr[$low]) {
        return $low;
    } 
    
    return -1;
    
}
```



插值搜索需要更多的计算步骤，但是如果数据是均匀分布的，这个算法的平均复杂度是O(log(log n))，这比二分搜索的复杂度O(logn)要好得多。 此外，如果值的分布不均匀，我们必须要小心。 在这种情况下，插值搜索的性能可以需要重新评估。下面我们将探索另一种称为指数搜索的二分搜索变体。



#### 指数搜索

在二分搜索中，我们在整个列表中搜索给定的数据。指数搜索通过决定搜索的下界和上界来改进二分搜索，这样我们就不会搜索整个列表。它减少了我们在搜索过程中比较元素的数量。指数搜索是在以下两个步骤中完成的：

1. 我们通过查找第一个指数k来确定边界大小，其中值2^k的值大于搜索项。 现在，2^k和2^(k-1)分别成为上限和下限。
2. 使用以上的边界来进行二分搜索。

下面我们来看下PHP实现的代码

```php
function exponentialSearch(array $arr, int $needle): int {
  $length = count($arr);
  if ($length == 0) return -1;
  $bound = 1;
  while($bound < $length; && $arr[$bound] < $needle) {
    $bound *= 2;
  }
  
  return binarySearch($arr, $needle, $bound >> 1, min($bound, $length));
}
```

我们把$needle出现的位置记位i，那么我们第一步花费的时间复杂度就是O(logi)。表示为了找到上边界，我们的while循环需要执行O(logi)次。因为下一步应用一个二分搜索，时间复杂度也是O(logi)。我们假设j是我们上一个while循环执行的次数，那么本次二分搜索我们需要搜索的范围就是2^j-1 至 2^j，而j=logi，即

![img](https://pic2.zhimg.com/80/v2-6e9e09350f2c075e51d082eb3cd3458d_1440w.png)

那我们的二分搜索时间复杂度需要对这个范围求log2，即

![img](https://pic3.zhimg.com/80/v2-d4ff3253c8827d30be5b13137c07ed46_1440w.png)

那么整个指数搜索的时间复杂度就是2 O(logi)，省略掉常数就是O(logi)。

![img](https://pic3.zhimg.com/80/v2-7fe057bedee293a411a7a3d52bc26a56_1440w.jpg)

#### 哈希查找

在搜索操作方面，哈希表可以是非常有效的数据结构。在哈希表中，每个数据都有一个与之关联的唯一索引。如果我们知道要查看哪个索引，我们就可以非常轻松地找到对应的值。通常，在其他编程语言中，我们必须使用单独的哈希函数来计算存储值的哈希索引。散列函数旨在为同一个值生成相同的索引，并避免冲突。

PHP底层C实现中数组本身就是一个哈希表，由于数组是动态的，不必担心数组溢出。我们可以将值存储在关联数组中，以便我们可以将值与键相关联。

```php
function hashSearch(array $arr, int $needle)
{
    return isset($arr[$needle]) ? true : false;
}
```

#### 树搜索

搜索分层数据的最佳方案之一是创建搜索树。在第[理解和实现树](https://link.zhihu.com/?target=https%3A//segmentfault.com/a/1190000015635928)中，我们了解了如何构建二叉搜索树并提高搜索效率，并且介绍了遍历树的不同方法。 现在，继续介绍两种最常用的搜索树的方法，通常称为广度优先搜索（BFS）和深度优先搜索（DFS）。

##### 广度优先搜索（BFS）

在树结构中，根连接到其子节点，每个子节点还可以继续表示为树。 在广度优先搜索中，我们从节点（主要是根节点）开始，并且在访问其他邻居节点之前首先访问所有相邻节点。 换句话说，我们在使用BFS时必须逐级移动。

![img](https://pic3.zhimg.com/80/v2-deb80829df1d0d51da6a66db13f4761e_1440w.jpg)

使用BFS，会得到以下的序列。

![img](https://pic1.zhimg.com/80/v2-a8cae51654cc6103cbd741030a254bac_1440w.png)

伪代码如下：

```
procedure BFS(Node root)
    Q := empty queue
    Q.enqueue(root)
    
    while(Q != empty) 
        u := Q.dequeue()
        for each node w that is childnode of u
            Q.enqueue(w)
        end for each
    end while
end procedure
```

下面是PHP代码。

```php
class TreeNode
{
  public $data = null;
  public $children = [];
  
  public function __construct(string $data = null) {
    $this->data = $data;
  }
  
  public function addChildren(TreeNode $treeNode) {
    $this->children[] = $treeNode;
  }
}

class Tree
{
  public $root = null;
  
  public function __construct(TreeNode $treeNode) {
    $this->root = $treeNode;
  }
  public function BFS(TreeNode $node): array {
    $arr = [];
    $visited = [];
    array_unshift($arr, $node);
    while (!empty($arr)) {
      $current = array_shift($arr);
      array_unshift($visited, $current);
      
      foreach ($current->children as $children) {
        array_unshift($arr, $children);
      }
    }
    return $visited;
  }
  
}
```

如果想要查找节点是否存在，可以为当前节点值添加简单的条件判断即可。BFS最差的时间复杂度是O（|V| + |E|），其中V是顶点或节点的数量，E则是边或者节点之间的连接数，最坏的情况空间复杂度是O（|V|）。

图的BFS和上面的类似，但略有不同。 由于图是可以循环的（可以创建循环），需要确保我们不会重复访问同一节点以创建无限循环。 为了避免重新访问图节点，必须跟踪已经访问过的节点。可以使用队列，也可以使用图着色算法来解决。



##### 深度优先搜索（DFS）

深度优先搜索（DFS）指的是从一个节点开始搜索，并从目标节点通过分支尽可能深地到达节点。 DFS与BFS不同，简单来说，就是DFS是深入挖掘而不是先扩散。DFS在到达分支末尾时然后向上回溯，并移动到下一个可用的相邻节点，直到搜索结束。还是上面的树

![img](https://pic2.zhimg.com/80/v2-3ac8b581e62926bd15b97dca768849fd_1440w.jpg)

这次我们会获得不通的遍历顺序(中序遍历)：

![img](https://pic1.zhimg.com/80/v2-af9254dd0f5ac15935de2472ee315be4_1440w.png)

从根开始，然后访问第一个孩子，即3。然后，到达3的子节点，并反复执行此操作，直到我们到达分支的底部。在DFS中，我们将采用递归方法来实现(中序)。

```
procedure DFS(Node current)
    for each node v that is childnode of current
       DFS(v)
    end for each
end procedure
```



```php
public function DFS(TreeNode $node): array {
  $visited[] = $node;
  if ($node->children) {
    foreach($node->children as $children) {
      array_unshift($visited, $children);
    }
  }
  return $visited;
}
```

如果需要使用迭代实现，必须记住使用栈而不是队列来跟踪要访问的下一个节点。下面使用迭代方法的实现

```php
public function DFS(TreeNode $node): SplQueue
{
    $stack = new SplStack();
    $visited = new SplQueue();

    $stack->push($node);

    while (!$stack->isEmpty()) {
        $current = $stack->pop();
        $visited->enqueue($current);

        foreach ($current->children as $child) {
            $stack->push($child);
        }
    }

    return $visited;
}
```

这看起来与BFS算法非常相似。主要区别在于使用栈而不是队列来存储被访问节点。它会对结果产生影响。上面的代码将输出8 10 14 13 3 6 7 4 1。这与我们使用迭代的算法输出不同，但其实这个结果没有毛病。

因为使用栈来存储特定节点的子节点。对于值为8的根节点，第一个值是3的子节点首先入栈，然后，10入栈。由于10后来入栈，它遵循LIFO。所以，如果我们使用栈实现DFS，则输出总是从最后一个分支开始到第一个分支。可以在DFS代码中进行一些小调整来达到想要的效果。

```php
public function DFS(TreeNode $node): SplQueue
{
    $stack = new SplStack();
    $visited = new SplQueue();

    $stack->push($node);

    while (!$stack->isEmpty()) {
        $current = $stack->pop();
        $visited->enqueue($current);

        $current->children = array_reverse($current->children);
        foreach ($current->children as $child) {
            $stack->push($child);
        }
    }

    return $visited;
}
```

由于栈遵循Last-in，First-out（LIFO），通过反转，可以确保先访问第一个节点，因为颠倒了顺序，栈实际上就作为队列在工作。要是我们搜索的是二叉树，就不需要任何反转，因为我们可以选择先将右孩子入栈，然后左子节点首先出栈。

DFS的时间复杂度类似于BFS。



### 树的遍历

总结
前序：根左右；中序：左根右；后序：左右根； 

#### 首先给出二叉树节点类

```php
class TreeNode {
    public $val;
    //左子树
    public  $left;
    //右子树
    public 	$right;
    //构造方法
    function __construct($x) {
       $this->val = $x;
    }
}
```



#### 1. 前序遍历

​	首先访问根节点，然后遍历左子树，最后遍历右子树

![1631025507913](algorithm.assets/1631025507913.png)

##### 递归实现前序遍历

先输出节点的值，再递归遍历左右子树

```php
public function recursionTree($tree) {
  	// 判断节点是否为空
  	if ($root != null) {
         	echo $tree->val . ' '; // 输出结果
      		recursionTree($tree->left); //遍历 左节点b
      		recursionTree($tree->right); // 右节点遍历
    }
}
```

##### 非递归前序遍历

因为要在遍历完节点的左子树后接着遍历节点的右子树，为了能找到该节点，需要使用**栈**来进行暂存。中序和后序也都涉及到回溯，所以都需要用到**栈**。

##### ![img](algorithm.assets/webp-20211102230743124)

```php
public function preorderTree($tree) {
  	// 定义一个暂时存放节点的数组，模仿栈
  	$arr = [];
  	// 新建游标节点为跟节点
 		$node = $tree;
  	// 当遍历最后一个节点的时候，无论它的左右子树都为空，并且存放节点的数组也为空
  	// 所以，只要不同时满足这两点，都需要进入循环
  	while ($node != null || !empty($arr)) {
      	// 当前节点非空，输出值
      	// 由于遍历顺序得知，需要一直往左走
      	while ($node != null) {
          	echo $node->val . " ";
          	// 为了之后找到该节点的右子树，暂存该节点
          	array_push($arr, $node);
          	$node = $node->left;
        }
      
      	// 一直到左子树为空，则开始遍历右子树
      	// 如果数组为空，就不需要在考虑
      	// 弹出数组(模仿的栈)头部元素，将游标等于改节点的右子树
      	if (!empty($arr)) {
          	$node = array_pop($arr);
          	$node = $node->right;
        }
    }
}
```



#### 2. 中序遍历

 先遍历左子树，然后访问根节点，然后遍历右子树。 

![1631025609029](algorithm.assets/1631025609029.png)

##### 递归实现中序遍历

```php
public function recursionMiddleTree($tree) {
  	if ($tree != null) {
      		recursionMiddleTree($tree->left)
         	echo $tree->val . ' ';
      		recrusionMiddleTree($tree->right);
    }
}
```

##### 非递归实现中序遍历

```php
public function recursionMiddleTree($tree) {
  	// 定义一个数组，模仿实现栈
  	$arr = [];
  	// 先建一个游标指向跟节点
  	$node = $tree;
  	// 当节点不为空，或者栈数组不为空时，继续遍历
  	while ($node != null || !empty($arr)) {
      	// 先遍历左子树，将其放到栈数组中，直到其为空
      	while ($node != null) {
          	// 将当前节点放入到栈数组中，方便之后遍历右子树
          	array_push($arr, $node);
          	$node = $node->left;
        }
      
      	// 遍历右子树
      	if (!empty($node)) {
          	// 出栈
          	$node = array_pop($arr);
          	echo $node->val . ' ';
          	$node = $node->right;
        }
    }
}
```





#### 3. 后序遍历

 先遍历左子树，然后遍历右子树，最后访问树的根节点。 

![1631025757239](algorithm.assets/1631025757239.png)

##### 递归实现后序遍历

```php
public function recursionTree($tree) {
  	if ($tree != null) {
      	recursionTree($tree->left);
      	recursionTree($tree->right);
      	echo $tree->val . ' ';
    }
}
```

##### 非递归实现后序遍历

后续遍历和先序、中序遍历不太一样。

后序遍历在决定是否可以输出当前节点的值的时候，需要考虑其左右子树是否都已经遍历完成。

所以需要设置一个**lastVisit游标**。

若lastVisit等于当前考查节点的右子树，表示该节点的左右子树都已经遍历完成，则可以输出当前节点。

并把lastVisit节点设置成当前节点，将当前游标节点node设置为空，下一轮就可以访问栈顶元素。

**否者，需要接着考虑右子树，node = node.right。**

以下考虑后序遍历中的三种情况：

![img](algorithm.assets/webp-20211102230839309)

​																										

如上图所示，从节点1开始考查直到节点4的左子树为空。

注：此时的游标节点node = 4.left == null。

此时需要从栈中**查看 array**栈顶元素。

发现节点4的右子树非空，需要接着考查右子树，4不能输出，node = node.right。



![img](algorithm.assets/webp-20211102230946328)



如上图所示，考查到节点7(7.left == null，7是从栈中弹出)，其左右子树都为空，可以直接输出7。

此时需要把lastVisit设置成节点7，并把游标节点node设置成null，下一轮循环的时候会考查栈中的节点6。



![img](algorithm.assets/webp-20211102231059825)

如上图所示，考查完节点8之后(lastVisit == 节点8)，将游标节点node赋值为栈顶元素6，节点6的右子树正好等于节点8。表示节点6的左右子树都已经遍历完成，直接输出6。

此时，可以将节点直接从栈中弹出Pop()，之前用的只是array[0]。

将游标节点node设置成null。

```php
public function postorderTree($tree) {
  	// 创建一个栈数组，用于存放节点信息
  	$arr = [];
  	// 游标节点指向跟节点
  	$node = $tree;
  	// 后序遍历在决定是否可以输出当前节点的值的时候，需要考虑其左右子树是否都已经遍历完成。所以需要设置一个lastVisit游标。
  	$lastVisit = $tree;
  	
  	while ($node != null || !empty($arr)) {
      	// 左子树遍历,并将节点存入栈数组
      	while ($node != null) {
          	array_push($arr, $node);
          	$node = $node->left;
        }
      
      	// 查看当前栈顶元素
      	$node = $arr[0];
      	
      	// 如果其右子树也为空，或者右子树已经访问
      	// 则可以输出当前节点的值
      	if ($node->right == null || $node->right = $lastVisit) {
          	echo $node->val . " ";
          	$node = array_pop($arr);
          	$lastVisit = $node;
          	$node = null;
        } else {
          	// 否则，继续遍历右子树
          	$node = $node->right;
        }
    }
}
```



#### 二叉树的前序遍历

![1631026333052](algorithm.assets/1631026333052.png)

```bash
输入：root = [1,null,2,3]
输出：[1,2,3]
```

![1631026416740](algorithm.assets/1631026416740.png)

```bash
输入：root = [1,2]
输出：[1,2]
```

![1631026443755](algorithm.assets/1631026443755.png)



```bash
输入：root = [1,null,2]
输出：[1,2]
```





![1630933752649](algorithm.assets/1630933752649.png)

### 冒泡算法

算法描述

- 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
- 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
- 针对所有的元素重复以上的步骤，除了最后一个；
- 重复步骤1~3，直到排序完成。

```php
function bubbleSort(array $arr) :array {
    $len = count($arr);
    for ($i = 0; $i < $len - 1; $i++) {
        for ($j = $i + 1; $j < $len - 1; $j++) {
            if ($arr[$i] > $arr[$j]) {
                $tmp = $arr[$i];
                $arr[$i] = $arr[$j];
                $arr[$j] = $tmp;
            }
        }
    }
    return $arr;
}
```

###  选择排序

 原理：首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。  

 算法描述

n个记录的直接选择排序可经过n-1趟直接选择排序得到有序结果。具体算法描述如下：

- 初始状态：无序区为R[1..n]，有序区为空；
- 第i趟排序(i=1,2,3…n-1)开始时，当前有序区和无序区分别为R[1..i-1]和R(i..n）。该趟排序从当前无序区中-选出关键字最小的记录 R[k]，将它与无序区的第1个记录R交换，使R[1..i]和R[i+1..n)分别变为记录个数增加1个的新有序区和记录个数减少1个的新无序区；
- n-1趟结束，数组有序化了。

```php
function selectSort(array $arr) :array {
    $len = count($arr);
    $minIndex = 0; $tmp = 0;
    for ($i = 0; $i < $len - 1; $i++) {
        $minIndex = $i;
        for ($j = $i + 1; $j < $len; $j++) {
            if ($arr[$j] < $arr[$minIndex]) {
                $minIndex = $j;
            }
        }
        $tmp = $arr[$i];
        $arr[$i] = $arr[$minIndex];
        $arr[$minIndex] = $tmp;
    }
    return $arr;
}
```

### 插入排序

 原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。 

算法描述

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：

- 从第一个元素开始，该元素可以认为已经被排序；
- 取出下一个元素，在已经排序的元素序列中从后向前扫描；
- 如果该元素（已排序）大于新元素，将该元素移到下一位置；
- 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置；
- 将新元素插入到该位置后；
- 重复步骤2~5。

```php
function insertSort(array $arr) :array {
    $len = count($arr);
    $preIndex = 0; $current = 0;
    for ($i = 1; $i < $len; $i++) {
        $preIndex = $i - 1;
        current = $arr[$i];
        while ($preIndex >= 0 && $arr[$preIndex] > $current) {
            $arr[$preIndex + 1] = $arr[$preIndex];
            $preIndex--;
        }
        $arr[$preIndex + 1] = $current;
    }
    return $arr;
}
```

### 希尔排序

 是简单插入排序的改进版。它与插入排序的不同之处在于，它会优先比较距离较远的元素。希尔排序又==缩小增量排序==。 

算法描述

先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，具体算法描述：

- 选择一个增量序列t1，t2，…，tk，其中ti>tj，tk=1；
- 按增量序列个数k，对序列进行k 趟排序；
- 每趟排序，根据对应的增量ti，将待排序列分割成若干长度为m 的子序列，分别对各子表进行直接插入排序。仅增量因子为1 时，整个序列作为一个表来处理，表长度即为整个序列的长度。

```php
function shellSort(array $arr) :array {
    $len = $num = count($arr);
   do {
       $step = $num = intval($num/2);
       // 对每组进行插入排序，将一个记录插入到已排序好的序列中，从而得到一个新的有序序列
       for ($i = $step; $i < $length; $i++) {
           if ($arr[$i] < $arr[$i - $step]) {
               $min = $arr[$i]; // 保存小的数
               for ($j = $i - $step; $j >= 0 && $min <$arr[$j]; $j-=$step) {
                   // 往后排
                   $arr[$j+ $step] = $arr[$j];
               }
               $arr[$j + $step] = $min;
           }
       }
   } while ($step > 1);
    return $arr;
}
```

### 快速排序

 快速排序的基本思想：通过一趟排序将待排记录分隔成独立的两部分，其中一部分记录的关键字均比另一部分的关键字小，则可分别对这两部分记录继续进行排序，以达到整个序列有序。 

算法描述

快速排序使用分治法来把一个串（list）分为两个子串（sub-lists）。具体算法描述如下：

- 从数列中挑出一个元素，称为 “基准”（pivot）；
- 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区退出之后，该基准就处于数列的中间位置。这个称为分区（partition）操作；
- 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

```php

```

### 存在重复元素

给定一个整数数组，判断是否存在重复元素。

如果存在一值在数组中出现至少两次，函数返回 `true` 。如果数组中每个元素都不相同，则返回 `false` 。

示例 1:

输入: [1,2,3,1]
输出: true

示例 2:

输入: [1,2,3,4]
输出: false

示例 3:

输入: [1,1,1,3,3,4,3,2,4,2]
输出: true

```php
function containsDuplicate(array $nums) {
   $len = count($nums);
    $tmp = [];
   for ($i = 0; $i < $len - 1; $i++) {
       if (isset($tmp[$arr[$i]])) return true;
       $tmp[$arr[$i]] = $i;
   }
    return false;
}
```

### 最大子序和

 给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。 

示例 1：

输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
示例 2：

输入：nums = [1]
输出：1
示例 3：

输入：nums = [0]
输出：0
示例 4：

输入：nums = [-1]
输出：-1
示例 5：

输入：nums = [-100000]
输出：-100000


提示：

1 <= nums.length <= 3 * 104
-105 <= nums[i] <= 105


进阶：如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的 分治法 求解。

```php
// 贪心算法
function maxSubArray($nums) {
    $n = count($nums);
    $curSum = $max = $nums[0];
    for ($i = 1; $i < $n; $i++) {
        if ($curSum < 0) {
            $curSum = $nums[$i];
        } else {
            $curSum += $nums[$i];
        }
        $max = max($max, $curSum);
    }
    return $max;
}
```

### 数字的补数

对整数的二进制表示取反（0 变 1 ，1 变 0）后，再转换为十进制表示，可以得到这个整数的补数。

例如，整数 5 的二进制表示是 "101" ，取反后得到 "010" ，再转回十进制表示得到补数 2 。
给你一个整数 num ，输出它的补数。

**示例 1：**

```
输入：num = 5
输出：2
解释：5 的二进制表示为 101（没有前导零位），其补数为 010。所以你需要输出 2 。
```

**示例 2：**

```
输入：num = 1
输出：0
解释：1 的二进制表示为 1（没有前导零位），其补数为 0。所以你需要输出 0 。
```

算法：

1. 通过右移1位来遍历二进制数位数。
2. 遍历的同时，定义一个二进制数$res，每位赋值1。
3. 按位异或，\$res ^ $num就有取反的效果。

```php
class Solution {

    /**
     * @param Integer $num
     * @return Integer
     */
    function findComplement($num) {
        $tmp = $num;
        $res = 0;
        while ($tmp != 0) {
            $res = ($res << 1) + 1;
            $tmp >>= 1;
        }

        return $num ^ $res;
    }
}
```

### 两数之和

给定一个整数数组 nums 和一个整数目标值 target，请你在该数组中找出 和为目标值 target  的那 两个 整数，并返回它们的数组下标。

你可以假设每种输入只会对应一个答案。但是，数组中同一个元素在答案里不能重复出现。

你可以按任意顺序返回答案。

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[0,1]
解释：因为 nums[0] + nums[1] == 9 ，返回 [0, 1] 。
```

**示例 2：**

```
输入：nums = [3,2,4], target = 6
输出：[1,2]

```

**示例 3：**

```
输入：nums = [3,3], target = 6
输出：[0,1]
```

```php
class Solution {

    /**
     * @param Integer[] $nums
     * @param Integer $target
     * @return Integer[]
     */
    function twoSum($nums, $target) {
        $map=[];//哈希查找表
        foreach($nums as $key=>$item){
            $b=$target-$item;
            if(isset($map[$b])){
                return [$map[$b],$key];//找到返回
            }else{
                $map[$item]=$key;//放入哈希表
            }
        }
    }
}

```



### 整数反转

给你一个 32 位的有符号整数 x ，返回将 x 中的数字部分反转后的结果。

如果反转后整数超过 32 位的有符号整数的范围 [−231,  231 − 1] ，就返回 0。

假设环境不允许存储 64 位整数（有符号或无符号）。

**示例 1：**

```
输入：x = 123
输出：321


```

**示例 2：**

```
输入：x = -123
输出：-321

```

**示例 3：**

```
输入：x = 120
输出：21

```

**示例 4：**

```
输入：x = 0
输出：0
```

```php
/**
 * @param Integer $x
 * @return Integer
 */
function reverse($x) {
   if (!is_int($x)) return 0;
    $res = 0;
    $max = pow(2, 31) - 1;
    $min = pow(-2, 31);
    while ($x != 0) {
        $remainder = $x % 10;
        $x = ($x - $remainder) / 10;
        $res = $res * 10 + $remainder;
    }
    if ($res > $max) return 0;
    if ($res < $min) return 0;
    return $res;

}

```



### 回文数

给你一个整数 x ，如果 x 是一个回文整数，返回 true ；否则，返回 false 。

回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。例如，121 是回文，而 123 不是。

**示例 1：**

```
输入：x = 121
输出：true
```

**示例 2：**

```
输入：x = -121
输出：false
解释：从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```

**解题思路**

​	转字符串

```php
/**
     * @param Integer $x
     * @return Boolean
     */
    function isPalindrome($x) {
        if ($x < 0 ) return false;
        
        $str = strval($x);
        $len = strlen($str);
        $k   = 0;

        while ($k < $len / 2) {
            if ($str[$k] !== $str[$len - $k - 1]) return false;
            $k++;
        }

        return true;
    }


```



### 罗马数字转整数

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

```
字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
```

例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

​	I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
​	X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
​	C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。
给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

**示例 1:**

```
输入: "III"
输出: 3
```

**示例 2:**

```
输入: "IV"
输出: 4
```

**示例 3:**

```
输入: "IX"
输出: 9
```

**示例 4:**

```
输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```

**示例 5:**

```
输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```

```php
/**
     * @param String $s
     * @return Integer
     */
    function romanToInt($s) {
        $arr = ['I' => 1, 'V' => 5, 'X' => 10, 'L' => 50, 'C' => 100, 'D' => 500, 'M' => 1000];
        $a = str_split($s);
        $r = 0;
        $p = 0;
        foreach($a as $k) {
            if ($p && $arr[$k] > $p) {
              // IV = I + V - I * 2 , 为什么是$p * 2 的原因
                $r += $arr[$k] - $p * 2;
            } else {
                $r += $arr[$k];
            }
            $p = $arr[$k];
        }
        return $r;
    }

```



### 最长公共前缀

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 `""`。

**示例 1：**

```
输入：strs = ["flower","flow","flight"]
输出："fl"
```

**示例 2：**

```
输入：strs = ["dog","racecar","car"]
输出：""
解释：输入不存在公共前缀。
```

```php
/**
 * @param String[] $strs
 * @return String
 */
function longestCommonPrefix($strs) {
  $commonPre = '';
	if (empty($strs)) return $commonPre;
  
	if (! isset($strs[1])) return $strs[0];
  
  // 对数组降序排序,SORT_STRING - 单元被作为字符串来比较
	rsort($strs, SORT_STRING);
  
  // 取数组两头，排序之后差异最大
	$first_ele = array_shift($strs);
	$last_ele = array_pop($strs);
  
	$len = strlen($first_ele);
	for ($i = 0; $i < $len; ++$i) {
		if ($first_ele[$i] != $last_ele[$i]) break;
		$commonPre .= $first_ele[$i];
	}
	return $commonPre;
}

```



### 有效的括号

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。

**示例 1：**

```
输入：s = "()"
输出：true
```

**示例 2：**

```
输入：s = "()[]{}"
输出：true
```

**示例 3：**

```
输入：s = "(]"
输出：false
```

**示例 4：**

```
输入：s = "([)]"
输出：false
```

**示例 5：**

```
输入：s = "{[]}"
输出：true
```



遍历整个字符串，遇到左括号就入栈，然后遇到和栈顶对应的右括号就出栈，遍历结束后，如果栈为空，就表示全部匹配。

```php
  /**
     * @param String $s
     * @return Boolean
     */
    function isValid($s) {
        $map = [
            ")" => "(",
            "}" => "{",
            "]" => "[",
        ];

        $len = strlen($s);
        $stack = [];

        //s中出现map的key则弹出，没有出现则入栈
        for ($i =0; $i<$len; $i++) {
            if (isset($map[$s[$i]])){
                //s中出现map的key：如果能找到对应的map的值 (,{,[ 则说明有配对，则弹出
                if (isset($stack)  && $stack[0] == $map[$s[$i]]) {
                    array_shift($stack);
                } else { //仅找到后面的一部分，说明是不匹配的
                    return false;
                }
            } else {
                array_unshift($stack, $s[$i]);
            }
        }

        if (count($stack) > 0) {
            return false;
        }

        return true;
    }
```



### 合并两个链表

将两个升序链表合并为一个新的 **升序** 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

**示例 1：**

![img](algorithm.assets/merge_ex1.jpg)

```
输入：l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
```

**示例 2：**

```
输入：l1 = [], l2 = []
输出：[]
```

**示例 3：**

```
输入：l1 = [], l2 = [0]
输出：[0]
```

**非递归解法**

```php
function mergeTwoLists($l1, $l2)
{
    $dummyHead = new ListNode(null);
    $cur = $dummyHead;
    while ($l1 !== null && $l2 !== null) {
        if ($l1->val <= $l2->val) {
            $cur->next = $l1;
            $l1 = $l1->next;
        } else {
            $cur->next = $l2;
            $l2 = $l2->next;
        }
        $cur = $cur->next;
    }

    if ($l1 !== null) {
        $cur->next = $l1;
    } elseif ($l2 !== null) {
        $cur->next = $l2;
    }

    return $dummyHead->next;
}

```

**递归解法**

```php
function mergeTwoLists($l1, $l2)
{
    // 递归解法
    // 递归函数的含义：返回当前两个链表合并之后的头节点(每一层都返回排序好的链表头)
    if ($l1 === null) return $l2;
    if ($l2 === null) return $l1;

    if ($l1->val < $l2->val) {
        $l1->next = $this->mergeTwoLists($l1->next, $l2);
        return $l1;
    } else {
        $l2->next = $this->mergeTwoLists($l1, $l2->next);
        return $l2;
    }
}

```



### 删除有序数组中的重复项

给你一个有序数组 nums ，请你 原地 删除重复出现的元素，使每个元素 只出现一次 ，返回删除后数组的新长度。

不要使用额外的数组空间，你必须在 原地 修改输入数组 并在使用 O(1) 额外空间的条件下完成。

说明:

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```
// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```



**示例 1**：

```
输入：nums = [1,1,2]
输出：2, nums = [1,2]
解释：函数应该返回新的长度 2 ，并且原数组 nums 的前两个元素被修改为 1, 2 。不需要考虑数组中超出新长度后面的元素。
```

**示例 2：**

```
输入：nums = [0,0,1,1,1,2,2,3,3,4]
输出：5, nums = [0,1,2,3,4]
解释：函数应该返回新的长度 5 ， 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4 。不需要考虑数组中超出新长度后面的元素。
```

```php
   /**
     * @param Integer[] $nums
     * @return Integer
     */
    function removeDuplicates(&$nums) 
    {
        $n = count($nums);

        for ($i = $n - 1; $i > 0; --$i) {
            if ($nums[$i] == $nums[$i - 1]) {
                // echo 'delete i='. $i, PHP_EOL;
                unset($nums[$i]);
            }
        }
    }

```

```php
// 快慢指针解法
function removeDuplicates(&$nums) {
  $n = count($nums);
  $s = 0; $f = 1; // 两个指针
  while ($f < $n) {
    if ($nums[$s] != $nums[$f]) {
      $nums[++$s] = $nums[$f];
    }
    $f++;
  }
  return $s;
}
```



### 移除元素

给你一个数组 nums 和一个值 val，你需要 原地 移除所有数值等于 val 的元素，并返回移除后数组的新长度。

不要使用额外的数组空间，你必须仅使用 O(1) 额外空间并 原地 修改输入数组。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

说明:

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以「引用」方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:

```
// nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中 该长度范围内 的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

```

**示例 1：**

```
输入：nums = [3,2,2,3], val = 3
输出：2, nums = [2,2]
解释：函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。你不需要考虑数组中超出新长度后面的元素。例如，函数返回的新长度为 2 ，而 nums = [2,2,3,3] 或 nums = [2,2,0,0]，也会被视作正确答案。
```

**示例 2：**

```
输入：nums = [0,1,2,2,3,0,4,2], val = 2
输出：5, nums = [0,1,4,0,3]
解释：函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。注意这五个元素可为任意顺序。你不需要考虑数组中超出新长度后面的元素。
```

```php
 /**
     * @param Integer[] $nums
     * @param Integer $val
     * @return Integer
     */
    function removeElement(&$nums, $val) {        
        foreach($nums as $k => $v){
            if($v == $val ){
                unset($nums[$k]);
            }            
        }
        
        return count($nums);
    }

```



### 实现 strStr()

实现 strStr() 函数。

给你两个字符串 haystack 和 needle ，请你在 haystack 字符串中找出 needle 字符串出现的第一个位置（下标从 0 开始）。如果不存在，则返回  -1 。

**说明：**

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与 C 语言的 strstr() 以及 Java 的 indexOf() 定义相符。

**示例 1：**

```
输入：haystack = "hello", needle = "ll"
输出：2
```

**示例 2：**

```
输入：haystack = "aaaaa", needle = "bba"
输出：-1
```

**示例 3：**

```
输入：haystack = "", needle = ""
输出：0
```

```php
 function strStr($haystack, $needle) {
        if($needle == ''){return 0;}    // 空字符串返回0
        $len = strlen($haystack);
        $length = strlen($needle);
       
        $i = 0;$j = 0;
        while($i<$len && $j < $length){
            if($haystack[$i] == $needle[$j]){
                $i++;
                $j++;
            }else{
                $i = $i - $j + 1;
                $j = 0;
            }
            if($j == $length){
                return $i-$j;
            }
        }
        return -1;
    }

```



### 搜索插入位置

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

请必须使用时间复杂度为 `O(log n)` 的算法。

**示例 1:**

```
输入: nums = [1,3,5,6], target = 5
输出: 2
```

**示例 2:**

```
输入: nums = [1,3,5,6], target = 2
输出: 1
```

**示例 3:**

```
输入: nums = [1,3,5,6], target = 7
输出: 4
```

**示例 4:**

```
输入: nums = [1,3,5,6], target = 0
输出: 0
```

**示例 5:**

```
输入: nums = [1], target = 0
输出: 0
```

```php
public function searchInsert($nums, $target) {
        $n = count($nums);
        if ($n === 0) return 0;
        if ($target < $nums[0]) return 0;
        if ($target > end($nums)) return $n;

        $l = 0;
        $r = $n - 1;
        while ($l < $r) {
            $mid = $l + floor(($r - $l) / 2);
            if ($nums[$mid] === $target) return $mid;
            // 当中间元素严格小于目标元素时，肯定不是解
            if ($nums[$mid] < $target) {
                // 下一轮搜索区间是 [mid+1, right]
                $l = $mid + 1;
            } else {
                $r = $mid;
            }
        }

        return $l;
}
```



### 最后一个单词的长度

给你一个字符串 `s`，由若干单词组成，单词前后用一些空格字符隔开。返回字符串中最后一个单词的长度。

**单词** 是指仅由字母组成、不包含任何空格字符的最大子字符串。



**示例 1：**

```
输入：s = "Hello World"
输出：5
```

**示例 2：**

```
输入：s = "   fly me   to   the moon  "
输出：4
```

**示例 3：**

```
输入：s = "luffy is still joyboy"
输出：6
```

```php
function lengthOfLastWord($s) {
        // 下面这一行,有点偷懒了,直接生对内置函数^_^ 请忽略下面这行
        // return strlen(array_pop(explode(' ',rtrim($s))));
        // 万恶的上面一行
        if (empty($s)) return 0;
        $count = strlen($s);
        $len = 0;
        for ($i=$count-1;$i>=0;$i--) {
            if ($s[$i] != ' ') {
                $len++;
            }
            if ($len !=0 && $s[$i] == ' ') {
                break;
            }
        }
        return $len;
}
```



### 加一

给定一个由 整数 组成的 非空 数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

**示例 1：**

```
输入：digits = [1,2,3]
输出：[1,2,4]
解释：输入数组表示数字 123
```

**示例 2：**

```
输入：digits = [4,3,2,1]
输出：[4,3,2,2]
解释：输入数组表示数字 4321。
```

**示例 3：**

```
输入：digits = [0]
输出：[1]
```

```php
function plusOne($digits) {
    $len1 = count($digits);
    if ($len1 == 0) return [1];
    $carry = 0;
    $return = [];
    $i = $len - 1;
    // 直接在最后一位加上
    $digits[$i]++;
    if ($digits[$i] <= 9) return $digits;
    while ($i >= 0 || $carry) {
        $sum = $carry;
        if ($i >= 0) {
            $sum += $digits[$i];
            $i--;
        }

        $carry = floor($sum / 10);
        array_unshift($return, $sum % 10);
    }
    return $return;
}

```



### 二进制求和

给你两个二进制字符串，返回它们的和（用二进制表示）。

输入为 **非空** 字符串且只包含数字 `1` 和 `0`。

**示例 1:**

```
输入: a = "11", b = "1"
输出: "100"
```

**示例 2:**

```
输入: a = "1010", b = "1011"
输出: "10101"
```

```php
function addBinary($a, $b) {
    $len1 = strlen($a);
    $len2 = strlen($b);
    if ($len1 == 0) return $b;
    if ($len2 == 0) return $a;

    $return = '';
    $carry = 0;
    $i = $len1 - 1;
    $j = $len2 - 1;
    while ($i >= 0 || $j >= 0 || $carry) {
        $sum = $carry;
        if ($i >= 0) {
            $sum += substr($a, $i, 1);
            $i--;
        }

        if ($j >= 0) {
            $sum += substr($b, $j, 1);
            $j--;
        }
        
        // 进位处理，大于 2 就进一位
        $carry = $sum >= 2 ? 1 : 0;
        // 当前位剩余的只能是 0 或 1
        $return = ($sum & 1) . $return;
    }
    return $return;
}
```





### 回文链表

请判断一个链表是否为回文链表。

**示例 1:**

```php
输入: 1->2
输出: false
```

**示例 2:**

```php
输入: 1->2->2->1
输出: true
```

**解析：回文即正反序都一样，所以只要找出中间节点，然后翻转后面部分的链表，再一一进行比较**

**解法：**

**1.快慢指针找出中间节点，快指针每次走两个，慢指针走一格**

**2.翻转后边部分**

**3.一一比较，得出结果**

```php
// 链表节点类
class Node {
  public $data;
  public $next = null;
  
  public function __construct($data = null, $next = null) {
    	$this->data = $data;
    	$this->next = $next;
  }
}

class Solution {
  	// 判断是否是回文
  	function isPalindrome($head) {
      	// 假头节点
      	$dummyHead = new Node();
      	// 将头节点指向目标链表
      	$dummyHead->next = $head;
      	
      	// 快慢指针取中间位置
      	$centerNode = $doubleNode = $dummyHead;
      	while ($doubleNode->next) {
          	$centerNode = $centerNode->next;
          	$doubleNode = $doubleNode->next->next;
        }
      
      	// 反转后边部分
      	$preNode = null; // 前节点
      	$curNode = $centerNode->next; // 后部分的第一个节点
      	while ($curNode) {
          	$nextNode = $curNode->next; 
          	$curNode->next = $preNode;
          	$preNode = $curNode;
          	$curNode = $nextNode;
        }
          
      	// 一一比较
      $curNode = $dummyHead->next;
      // 遍历结束，￥preNode指向反转后，后面部分的第一个节点
      while ($curNode && $preNode) {
        	if ($curNode->val != $preNode->val) {
            	return false;
          }
        	$curNode = $curNode->next;
        	$preNode = $preNode->next;
      }
    	return true;
    }
}
```



### 重排链表

给定一个单链表 *L*：*L*0→L1→…→Ln-1→Ln ，
将其重新排列后变为： *L*0→Ln→L1→Ln-1→L2→Ln-2→…

你不能只是单纯的改变节点内部的值，而是需要实际的进行节点交换。

**示例 1:**   给定链表 1->2->3->4, 重新排列为 1->4->2->3.

**示例 2:**   给定链表 1->2->3->4->5, 重新排列为 1->5->2->4->3.

**解析：解法与234.回文链表类似，区别：这道题不是要比较，而是进行重新连接**

**解法：**

**1.快慢指针找出中间节点，快指针每次走两个，慢指针走一格**

**2.翻转后边部分，区分出两条单链表**

**3.一一链接，得出结果**

```php
// 链表节点类
class Node {
  public $data;
  public $next = null;
  
  public function __construct($data = null, $next = null) {
    	$this->data = $data;
    	$this->next = $next;
  }
}

class Solution
{
  	function reorderList($head) {
      	$dummyHead = new Node();
      	$dummyHead->next = $head;
      	
      	// 快慢指针取中间位置
      	$centerNode = $doubleNode = $dummyHead;
      	while ($doubleNode->next) {
          	$centerNode = $centerNode->next;
          	$doubleNode = $doubleNode->next->next;
        }
      
      	// 反转链表
      	$preNode = null;
      	$curNode = $centerNode-next;
      	while ($curNode) {
          	$nextNode = $curNode->next;
          	$curNode->next = $pr$eNode;
          	$preNode = $curNode;
          	$curNode = $nextNode;
        }
      
      	/**
      		*	将前部分的链表封尾，就完成构建两条链表
      		* head -> 1 -> 2 -> 3 -> end
      		* head -> 5 -> 4 -> end
      		*/
      	$centerNode->next = null;
      	// 两两合并，初始化两个链表的头部
      	$curNode = $dummyHead->next;
      	$revNode = $preNode;
      	while ($revVode) {
          	// 保存下一节点
          	$curNextNode = $curNode->next;
          	$revNextNode = $revNode->next;
          
          	// 交换节点
          	$curNode->next = $revNode;
          	$revNode->next = $cruNextNode;
          	
          	// 重新定义节点
          	$curNode = $curNextNode;
          	$revNode = $revNextNode;
        }
      return $dummyHead->next;
    }
}
```

### 将 1234567890转换成1,234,567,890，每三位用逗号隔开

思路：

1. 翻转字符，将数字变成0987654321, 

2. 然后隔开 098,765,432,1
3. 在翻转回来

```php
function str($str) {
    // 翻转字符
    $str = strrev($str);
    // 分割字符
    $str = chunk_split($str, 3, ",");
    // 翻转回来
    $str = strrev($str);
    // 去掉左侧逗号
    $str = ltrim($str, ",");
    return $str;
}
```



### 猴子选大王问题

一群猴子排成一圈，按1,2，。。。，n依次编号.然后从第1只开始数，数到第m只，把它踢出圈，从它后面再开始数，再数到第m只，再把它踢出去。。。，如此不停的进行下去，直到最后一个猴子为止，那只猴子就叫做大王。要求编程模拟此过程，输入m、n，输出最后那个大王的编号。

```php
function hou_king($nm $m) {
    // 构造数组
    for ($i = 1; $i < $n; $i++) {
        $arr[] = $i;
    }
    $i = 0; // 设置数组指针
    // 猴子数量大于1进去循环
    while (count($arr) > 1) {
        // 判断猴子是否出局，如果出局就删掉，没出局放到数组最后，继续循环
        if (($i+1) % $m == 0) {
            unset($arr[$i]);
        } else {
            array_push($arr, $arr[$i]); // 把值加入数组末尾
            unset($arr[$i]); // 删除数组前面这个值
        }
        $i++;
    }
    return $arr;
}
```



### 求未出现的最小正整数

给定一组无序整数数组，找出其中未出现的最小正整数，例如[1,2,3,5]输出4

```php
/**
 * 思路1：把原理的数组去掉负数，重复数字，然后排序
 * 然后把数组下标+1跟值比较，找出第一个不同的，输出小标对应的值+1；全部相同，输出最大值加1
 */
function test($arr) {
    // 去重
    $arr = array_unique($arr);
    // 重新排一下下标
    $arr = array_merge($arr);
    $a = [];
    for ($i=0; $i < count($arr); $i++) {
        if ($arr[$i] > 0) {
			$a[] = $arr[$i];
        }
    }
    
    // 排序
    sort($a, SORT_NUMERIC);
    
    // 比较查找
    for ($i = 0; $i < count($a); $i++) {
        if ($a[$i] + 1 != $a[$i+1]) {
            return $a[$i] + 1;
        }
    }
    return -1;
}

// 思路2
// 直接死循环，一个个去尝试数组里面是否存在，如果第一个不存在的就输出结束
function test($arr) {
    for ($i=1; $i > 0; $i++) {
        if (!in_array($i, $arr)) {
            return $i;
        }
    }
    return -1;
}
```



### 文件锁机制

请写一段代码，确保多个进程同时写入同一个文件成功`

```php
// 思路： 加锁
$file = fopen("test.txt", "w+");
if (flock($file, LOCK_EX)) {
    // 获得写锁，开始写入数据
    fwrite($file, "xxx");
    // 打开锁
    flock($file, LOCK_UN);
} else {
    echo "file is locking";
}
// 关闭文件
fclose($file);
```

### 判断日期的合法性

```php
// 思路：先将日期转时间戳，再转回来，比较是否和原来的相同
function test($str) {
    if (date("Y-m-d H:i:s", strtotime($str)) == $str) {
        return true;
    }
    return false;
}
```



### 相对路径的计算

写一个函数，算出两个文件的相对路径，如$a = '/a/b/c/d/e.php', $b = '/a/b/12/34/.c.php';

计算出\$b相对于$a的相对路径应该是../../c/d

```php
// 思路
```





### 问题

##### **写一段代码，找到所有子集合，如 [a,b,c] 的子集合有 {},{a},{b},{c},{ab},{ac},{abc}**

##### **['a'=>200,'b'=>100,'c'=>100], 写一个自定义排序函数，按值降序，如果值一样，按键排序**

##### **设计一个缓存系统，可以定期或空间占满之后自动删除长期不用的数据，不能使用用遍历。**

##### **一个排序好的数组，将它从中间任意一个位置切分成两个数组，然后交换它们的位置并合并，合并后新数组元素如：20,21,22,25,30,1,2,3,5,6,7,8,15,18,19, 写一个查询函数来查找某个值是否存在。**

##### **设计一个树形结构，再写一个函数对它进行层序遍历**

