### 问题描述：

​	根据国际象棋的规则，皇后可以攻击与同处一行、一列或一条斜线上的棋子。给定 𝑛 个皇后和一个 𝑛×𝑛 大小的棋盘，寻找使得所有皇后之间无法相互攻击的摆放方案。

例如：

​	对于n = 4 的情况，有下面两种可能的摆放方法。

​	![1719151930499](images/N皇后/1719151930499.png)

![1719151962827](images/N皇后/1719151962827.png)

### 蛮力法：

​	蛮力法只需要将所有的情况都遍历到即可，对于每一行，尝试在每一列放置一个皇后，生成所有可能的放置方案。

​	每一行有n列，一共n行，遍历每种情况就需要$$n*n*n...*n = n^n$$次，再加上每一次都要判断所有的皇后位置是否正确，又需要遍历。所以复杂度特别高。

~~~ java
    public static void nQueens(int[][] data, int row) {
        int n = data.length;
        // 已经到达最后一行，n个皇后已经放置完毕
        if (row == n) {
            // 校验皇后拜访位置是否合理
            for (int row1 = 0; row1 < n; row1++) {
                for (int col = 0; col < n; col++) {
                    if (data[row1][col] == 1) {
                        // 判断每一行中皇后位置是否合理
                        if (!canPlace(data, row1, col)) {
                            // 不合理直接返回
                            return;
                        } else {
                            // 到达最后一行输出结果
                            if (row1 == n - 1) {
                                System.out.println("第" + ++num + "组解：");
                                for (int[] d : data) {
                                    System.out.println(Arrays.toString(d));
                                }
                            }
                        }
                    }
                }
            }
        }else{
            // 未到达最后一行，遍历
            for (int col = 0; col < n; col++) {
                data[row][col] = 1;
                nQueens(data, row + 1);
                data[row][col] = 0;
            }
        }
    }

    public static boolean canPlace(int[][] data, int row, int col) {
        // 检查同一列是否有皇后
        for (int i = 0; i < row; i++) {
            if (data[i][col] == 1) return false;
        }
        // 检查 \ 对角线是否存在皇后
        for (int i = 1; row - i >= 0 && col - i >= 0; i++) {
            if (data[row - i][col - i] == 1) return false;
        }
        // 检查 / 对角线是否存在皇后
        for (int i = 1; row - i >= 0 && col + i < data.length; i++) {
            if (data[row - i][col + i] == 1) return false;
        }
        return true;
    }
~~~



### 回溯法：

#### 1、定义问题并构造状态空间树

​	根据棋盘大小$$n*n$$和$$n$$个皇后，以及皇后可以攻击与其处于同一行上的其他皇后可知，每一行仅允许放置一个皇后。可以按照逐行放置的思路：从第一行开始，在每行放置一个皇后，直至最后一行结束。

​	那么每一行实际上就有n个选择，每个位置是否放入皇后，放入皇后之后，就可以进入下一行放置下一个皇后。

​	采用一个n叉树就可以表示，这里使用一个二维数组表示：

~~~ java
    public static void dfs(int[][] data, int row) {
        for (int col = 0; col < data.length; col++) {
                // 放入皇后
                data[row][col] = 1;
                // 放入下一个皇后
                dfs(data, row + 1);
                // 取出当前皇后(回溯):每一行只能放置一个，取出当前列皇后，下一列放入
                data[row][col] = 0;
            }
        }
    }
~~~

#### 2、选择和判断（剪枝）

​	在某一行的某一列放置皇后时，可能会出现如果该位置放置皇后，就会与前几行放置的皇后冲突，那么就可以提前剪枝，直接不用放置后续的皇后了，直接去尝试再下一列放置。

~~~ java
	public static void dfs(int[][] data, int row) {
        for (int col = 0; col < data.length; col++) {
            // 剪枝:判断该位置是否可以放入，不可放入则直接终止
            if (canPlace(data, row, col)) {
                // 放入皇后
                data[row][col] = 1;
                // 放入下一个皇后
                dfs(data, row + 1);
                // 取出当前皇后(回溯):每一行只能放置一个，取出当前列皇后，下一列放入
                data[row][col] = 0;
            }
        }
    }

	public static boolean canPlace(int[][] data, int row, int col) {
        // 检查同一列是否有皇后
        for (int i = 0; i < row; i++) {
            if (data[i][col] == 1) return false;
        }
        // 检查 \ 对角线是否存在皇后
        for (int i = 1; row - i >= 0 && col - i >= 0; i++) {
            if (data[row - i][col - i] == 1) return false;
        }
        // 检查 / 对角线是否存在皇后
        for (int i = 1; row - i >= 0 && col + i < data.length; i++) {
            if (data[row - i][col + i] == 1) return false;
        }
        return true;
    }
~~~

#### 3、输出结果

​	当放置到最后一行时，此时所有皇后均放入，可以输出结果

~~~ java
	public static void dfs(int[][] data, int row) {
        // 最后一个皇后已经放入
        if (row == data.length) {
            printResult(data, ++resultNum);
            return;
        }
        for (int col = 0; col < data.length; col++) {
            // 剪枝:判断该位置是否可以放入，不可放入则直接终止
            if (canPlace(data, row, col)) {
                // 放入皇后
                data[row][col] = 1;
                // 放入下一个皇后
                dfs(data, row + 1);
                // 取出当前皇后(回溯):每一行只能放置一个，取出当前列皇后，下一列放入
                data[row][col] = 0;
            }
        }
    }
	public static void printResult(int[][] data, int num) {
        System.out.println("第" + num + "组解：");
        for (int i = 0; i < data.length; i++) {
            System.out.println(Arrays.toString(data[i]));
        }
    }
~~~

#### 图解说明：

​	因为八皇后如果画图篇幅过大，这里用四皇后讲解：

​	其中白色表示尚未遍历，绿色表示放入皇后，红色表示被剪枝（此路不通）

1、给第一行第一列放入皇后，进入第二行在第二行第一列放入皇后，剪枝

![1719214619670](images/N皇后/1719214619670.png)

2、放入第二行第二列，剪枝

![1719214675446](images/N皇后/1719214675446.png)

3、放入第二行第三列，皇后可以放置，再尝试放入第三行

![1719215364717](images/N皇后/1719215364717.png)

4、第二行第三列放入皇后的所有情况均被剪枝，放入第二行第四列

​	放入第三行第一列时剪枝，放入第三行第二列

![1719215803447](images/N皇后/1719215803447.png)

5、第三行确定后，放入第四行（最后一个皇后）

​	可见，在第四行第三列放入皇后时，符合要求，直接输出结果，其他几种情况均不符合情况

![1719216294356](images/N皇后/1719216294356.png)

6、回溯，计算第三行放入第三列的情况

7、重复上述步骤

具体流程下图：

​	可以看到，这其实就是深度优先搜索，添加了剪枝

![1719217020972](images/N皇后/1719217020972.png)