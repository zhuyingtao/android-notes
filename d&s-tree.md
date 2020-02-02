### 二叉树

#### 存储方式

- 顺序存储
- 链式存储

#### 代码实现

- 二叉树的定义

  ```java
     /**
       * 设置结点结构
       */
      public static class TreeNode<T> {
          T val; // 二叉树的结点数据
          TreeNode<T> leftNode; // 二叉树的左子树（左孩子）
          TreeNode<T> rightNode; // 二叉树的右子树（右孩子）
  
          public TreeNode(T data,TreeNode<T> left,TreeNode<T> right) {
              this.val = data;
              this.leftNode = left;
              this.rightNode = right;
          }
  
  
          // 获得 & 设置二叉树的结点数据
          public T getData(){
              return val;
          }
  
          public void setData(T data){
              this.val = data;
          }
  
          // 获得 & 设置二叉树的左子树（左孩子）
          public TreeNode getLeftNode(){
              return leftNode;
          }
  
          public void setLeftNode(TreeNode leftNode){
              this.leftNode = leftNode;
          }
  
          // 获得 & 设置二叉树的右子树（右孩子）
          public TreeNode getRightNode(){
              return rightNode;
          }
          public void setRightNode(TreeNode rightNode){
              this.rightNode = rightNode;
          }
      }
  
  
  /**
   * 作用：构造二叉树
   * 注：必须逆序建立，即：先建立子节点，再逆序往上建立
   * 原因：非叶子节点会使用到下面的节点，而初始化是按顺序初始化的，不逆序建立会报错
   */ 
  public Node init(){
      // 结构如下：(由下往上建立)
      //            A
      //       B         C
      //    D         E     F
      //  G   H         I
      Node I = new Node("I", null, null);
      Node H = new Node("H", null, null);
      Node G = new Node("G", null, null);
      Node F = new Node("F", null, null);
      Node E = new Node("E", null, I);
      Node D = new Node("D", G, H);
      Node C = new Node("C", E, F);
      Node B = new Node("B", D, null);
      Node A = new Node("A", B, C);
      return A;  // 返回根节点
  }
  ```

- 二叉树的遍历

  - 前序遍历

    - 递归方式

    ```java
       /**
         * 内容：前序遍历
         * 方式：递归
         */
         public void preOrder(Node root){
            // 1. 判断二叉树结点是否为空；若是，则返回空操作
            if(root ==null)
                return;
    
            // 2. 访问根节点（显示根结点）
            printNode(root);
    
            // 3. 遍历左子树
            preOrder(root.getLeftNode());
    
            // 4. 遍历右子树
            preOrder(root.getRightNode());
    
        }
    ```

    - 非递归方式

      主要采用栈实现

    ```java
    /**
      * 方式：非递归（栈实现）
      */
        public static void preOrder_stack(Node root){
    
            Stack<Node> stack = new Stack<Node>();
    
            // 步骤1：直到当前结点为空 & 栈空时，循环结束
            while(root != null || stack.size()>0){
    
                // 步骤2：判断当前结点是否为空
                  // a. 若不为空，执行3
                  // b. 若为空，执行5
                  if(root != null){
    
                    // 步骤3：输出当前节点，并将其入栈
                    printNode(root);
                    stack.push(root);
    
                    // 步骤4：置当前结点的左孩子为当前节点
                    // 返回步骤1
                    root = root.getLeftNode();
    
                }else{
    
                    // 步骤5：出栈栈顶结点
                    root = stack.pop();
                    // 步骤6：置当前结点的右孩子为当前节点
                    root = root.getRightNode();
                      // 返回步骤1
                }
            }
        }
    ```

  - 中序遍历

    - 递归实现

    ```java
    /**
      * 方式：递归
      */
        public void InOrder(Node root){
        
            // 1. 判断二叉树结点是否为空；若是，则返回空操作
            if(root ==null)
                return;
    
            // 2. 遍历左子树
            InOrder(root.getLeftNode());
    
            // 3. 访问根节点（显示根结点）
            printNode(root);
    
            // 4. 遍历右子树
            InOrder(root.getRightNode());
    
        }
    ```

    - 非递归实现

      采用栈实现

    ```java
    /**
      * 方式：非递归（栈实现）
      */
        public static void InOrder_stack(Node root){
    
            Stack<Node> stack = new Stack<Node>();
    
            // 1. 直到当前结点为空 & 栈空时，循环结束
            while(root != null || stack.size()>0){
    
                // 2. 判断当前结点是否为空
                // a. 若不为空，执行3、4
                // b. 若为空，执行5、6
                if(root != null){
    
                    // 3. 入栈当前结点
                    stack.push(root);
    
                    // 4. 置当前结点的左孩子为当前节点
                    // 返回步骤1
                    root = root.getLeftNode();
    
                }else{
    
                    // 5. 出栈栈顶结点
                    root = stack.pop();
                    // 6. 输出当前节点
                    printNode(root);
                    // 7. 置当前结点的右孩子为当前节点
                    root = root.getRightNode();
                    // 返回步骤1
                }
            }
    ```

  - 后序遍历

    - 递归实现

    ```java
    /**
      * 方式：递归
      */
        public void PostOrder(Node root){
            // 1. 判断二叉树结点是否为空；若是，则返回空操作
            if(root ==null)
                return;
    
            // 2. 遍历左子树
            PostOrder(root.getLeftNode());
    
            // 3. 遍历右子树
            PostOrder(root.getRightNode());
    
            // 4. 访问根节点（显示根结点）
            printNode(root);
    
        }
    ```

    - 非递归实现

      采用栈实现

    ```java
    /**
      * 方式：非递归（栈实现）
      */
        public void PostOrder_stack(Node root){
    
            Stack<Node> stack = new Stack<Node>();
            Stack<Node> output = new Stack<Node>();
    
            // 步骤1：直到当前结点为空 & 栈空时，循环结束——> 步骤8
            while(root != null || stack.size()>0){
    
                // 步骤2：判断当前结点是否为空
                // a. 若不为空，执行3、4
                // b. 若为空，执行5、6
                if(root != null){
    
                    // 步骤3：入栈当前结点到中间栈
                    output.push(root);
    
                    // 步骤4：入栈当前结点到普通栈
                    stack.push(root);
    
                    // 步骤4：置当前结点的右孩子为当前节点
                    // 返回步骤1
                    root = root.getRightNode();
    
                }else{
    
                    // 步骤5：出栈栈顶结点
                    root = stack.pop();
                    // 步骤6：置当前结点的右孩子为当前节点
                    root = root.getLeftNode();
                    // 返回步骤1
                }
            }
    
            // 步骤8：输出中间栈的结点
            while(output.size()>0){
                printNode(output.pop());
            }
        }
    ```

  - 层序遍历

    非递归实现，采用队列

    ```java
    /**
      * 方式：非递归（采用队列）
      */
        public void levelTravel(Node root){
            // 创建队列
            Queue<Node> q=new LinkedList<Node>();
    
            // 1. 判断当前结点是否为空；若是，则返回空操作
            if(root==null)
                return;
            // 2. 入队当前结点
            q.add(root);
    
            // 3. 判断当前队列是否为空，若为空则跳出循环
            while(!q.isEmpty()){
    
                // 4. 出队队首元素
                root =  q.poll();
    
                // 5. 输出 出队元素
                printNode(root);
    
                // 6. 若出队元素有左孩子，则入队其左孩子
                if(root.getLeftNode()!=null) q.add(root.getLeftNode());
    
                // 7. 若出队元素有右孩子，则入队其右孩子
                if(root.getRightNode()!=null) q.add(root.getRightNode());
            }
        }
    
    ```

#### 二叉树类型

- 线索二叉树
- 二叉排序树
  - 平衡二叉排序树（AVL 树）





