![GitHub Logo](/image/22.1.png)

这题最开始的想法是用bfs.
难点是如何判断给定的string是否是合法的，有几点需要检查:

	1. "(" 最多只能有n 个
	2. ")" 最多只能有n 个
	3. "(" 作为开始的第一个字符
	4. 任何一个时间内不能有")" 在前面, "(" 在后边，也就是说不能有")"的个数大于"("

我的想法是定义一个Prenthese的类，这个里面会有一个add 方法每次加"("和")" 同时会有2个int 的value记录
左右括号的个数，这样每次通过这2个参数的检查就可以验证上面的4点， 详细code如下:

这个做法的优点是验证字符串是否合法比较快，只是比较左右括号的个数 O(1)就可以，
但是缺点是循环太多，尤其是queue 里要存储这个类，然后最后返回的时候要再loop出来转成List<String>格式，效率太差

```java
class Solution {
    public List<String> generateParenthesis(int n) 
    {        
        List<String> list = new ArrayList();
        if (n<=0) return list;
        Queue q = new LinkedList();    
        Prenthese pt = new Prenthese(n); 
        pt.add("(");
        q.offer(pt);
        
        for (int i=0;i<2*n-1;i++)
        {        
            int size = q.size();                        
            for (int j=0;j<size;j++)
            {                
                Prenthese ori = (Prenthese)q.poll();
                Prenthese newpt = null;
                try
                {
                    newpt = (Prenthese)ori.clone();
                }
                catch(Exception ex)
                {
                    
                }
                newpt.add("(");
                if (newpt.isValid())
                    q.offer(newpt);
                ori.add(")");
                if (ori.isValid())
                    q.offer(ori);
            }
        }
        
        int size = q.size();
        
        for (int i=0;i<size;i++)
        {
            list.add(((Prenthese)q.poll()).toString());
        }
        return list;
    }        
}

class Prenthese implements Cloneable 
{
    private int leftParenthese = 0;
    private int rightParenthese = 0;
    private int count = 0; 
    private String str = "";
    
    public Prenthese(int n)
    {
        this.count = n;
    }
    
    public void add(String str)
    {
        if ("(".equals(str))
        {
            leftParenthese++;
        }
        else if (")".equals(str))
        {
            rightParenthese++;
        }
        this.str+=str;        
    }
    
    public boolean isValid()
    {
        if (leftParenthese>count || rightParenthese>count || rightParenthese>leftParenthese)
            return false;
        else
            return true;
    }
        
    public String toString()
    {
        return this.str;
    }
    
    @Override
    public Object clone() throws CloneNotSupportedException 
    {
    	return super.clone();
    }    
}
```

所以我想了第2种做法，每次加新的左右括号的时候，能否用正则表达式去验证传入的字符是否合理呢，给出了下面的方案
```java
class Solution {
    public List<String> generateParenthesis(int n) 
    {                
        if (n<=0) return new ArrayList();
        Queue q = new LinkedList();
        q.add("(");
        
        for (int i=0; i<2*n-1; i++)
        {
            int size = q.size();
            for (int j=0;j<size;j++)
            {
                String tmp = (String)q.poll();
                int leftP = findNumberCount(tmp, "\\(");
                int rightP = findNumberCount(tmp, "\\)");
                if (isValid(n, leftP+1, rightP))
                    q.offer(tmp+"(");
                
                if (isValid(n, leftP, rightP+1))
                    q.offer(tmp+")");
            }
        }
        return new ArrayList(q);
    }        
    
    private boolean isValid(int n, int lParenthese, int rParenthese)
    {
        return (lParenthese<=n) && (rParenthese<=n) && (lParenthese>=rParenthese);
    }
    
    private int findNumberCount(String s , String findtext)
    {
        int count = 0;
        java.util.regex.Pattern p = java.util.regex.Pattern.compile(findtext);
        java.util.regex.Matcher m = p.matcher(s);
        while(m.find())
        {
            count++;
        }
        return count;
    }
}
```

这个做法不需要循环那么多次，但是缺点也很明显，即每次正则验证的时候，要把每个字符串从头到尾验证一遍，效率还是不高。

想了一下用hashmap是否能提高验证的速度呢，也就是把每个字符串的左右括号个数用一个数组存入hashmap，key就是字符串的值, 给出了下边的方案

```java
class Solution {
    public List<String> generateParenthesis(int n) 
    {                
        if (n<=0) return new ArrayList();
        Queue q = new LinkedList();
        q.add("(");        
        Map<String,Integer[]> map = new HashMap();
        Integer[] nums = new Integer[2];
        nums[0]=1;        
        map.put("(",nums);
        for (int i=0; i<2*n-1; i++)
        {
            int size = q.size();
            for (int j=0;j<size;j++)
            {
                String tmp = (String)q.poll();
                nums = map.get(tmp);
                if (nums==null)
                {
                	nums = new Integer[2];
                	nums[0] = 0;
                	nums[1] = 0;
                }
                Integer[] numsCopy = nums.clone();
                int leftP = nums[0]==null?0:nums[0];
                int rightP = nums[1]==null?0:nums[1];
                if (isValid(n, leftP+1, rightP))
                {
                    q.offer(tmp+"(");
                    nums[0]=(nums[0]==null?1:nums[0]+1);
                    map.put(tmp+"(",nums);
                }
                                    
                if (isValid(n, leftP, rightP+1))
                {
                    q.offer(tmp+")");
                    numsCopy[1]=(numsCopy[1]==null?1:numsCopy[1]+1);
                    map.put(tmp+")",numsCopy);
                }                    
            }
        }
        return new ArrayList(q);
    }        
    
    private boolean isValid(int n, int lParenthese, int rParenthese)
    {
        return (lParenthese<=n) && (rParenthese<=n) && (lParenthese>=rParenthese);
    }
    
}
```

发现效率还是不高，hashmap 也存在要遍历的问题(如果每个bucket比较长，转成了链表的话)

最后复习了一下backtracking ,发现用backtracking 是最快的, 验证也很简单，很容易验证上边的4点

```java
class Solution {
    public List<String> generateParenthesis(int n) 
    {                
        if (n<=0) return new ArrayList();
        List<String> list = new ArrayList();
        
        generateParenthesis(list, n, 0, 0, "");
        return list;
    }   
    
    public void generateParenthesis(List<String> list, int n, int leftP, int rightP, String str)
    {
        if (str.length()==2*n && leftP==n && rightP==n)
        {
            list.add(str);
            return;
        }                
        if (leftP<n)   
            generateParenthesis(list, n, leftP+1, rightP, str+"(");
        if (rightP<n && leftP>rightP)
            generateParenthesis(list, n, leftP, rightP+1, str+")");
    }
}

```