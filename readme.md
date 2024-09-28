```vue
<xycoder-threeStageSlidingPanel >
    ...内容
    </xycoder-threeStageSlidingPanel>
```

> 注意：父组件必须设置overflow: hidden;属性

![Image text](/static/gif.gif)
### 属性

| 属性名 | 默认值 | 说明 |
| :----: |  :----: |:----: |
| levelArray | ["0%", "20%", "70%"] |需要折叠的级别，必须为百分比，从小到大排序 |
| firstLevel | 70% |初次值的级别 |

### 属性

| 方法名 | 说明 |
| :----: |  :----: |
| foldPanel | 折叠面板 |

```javaScript
this.$refs.panel.foldPanel();
``` 
