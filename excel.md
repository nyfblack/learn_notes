# 1.数据有效性校验

单击“数据”菜单——“有效性”命令，——“设置”卡；

## 1. 自定义公式

1. 使单元格区域内记录不能重复输入控制 　　

   =COUNTIF(A:A,A2)=1  (直接复制此公式进去即可)   　　

2. 禁止单元格输入数字控制 　　

   =ISNUMBER(A1)<>TRUE 

3. 允许单元格只能输入数字控制 　　

   =ISNUMBER(A1)=TRUE   　　

4. 禁止单元格输入字母和数字 　　

   =LENB(A1)=2   　　

5. 禁止输入周末日期　　

   =AND(WEEKDAY(A1)<>1,WEEKDAY(A1)<>7)   　　

6. 特定前缀输入:应该含某个字开头 　　

   =OR(LEFT(A1)="张",LEFT(A1)="李")   　　

7. 禁止单元格前后输入多余空格 　　

   =A1=TRIM(A1)   　　

8. 禁止输入数字大于某某值 　　

   =A1<=100   　　

9. 禁止输入限定的值　　

   =MAX(A:A)    同 <>""    同=""   　　

10. 限定区域输入的和的最大值 　　

    =SUM(A1:A10)<100 