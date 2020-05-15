+++
title = "hexo mermaid"
date = 2017-12-04
category = "Mermaid"

[taxonomies]
tags = ["mermaid"]
+++

### mermaid

#### flowchart

```mermaid
graph TD
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

#### sequence

```mermaid
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->>John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->>Alice: Great!
    John->>Bob: How about you?
    Bob-->>John: Jolly good!
```

#### gantt

```mermaid
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram to mermaid

section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d
```

#### git graph

```mermaid
gitGraph:
options
{
    "nodeSpacing": 150,
    "nodeRadius": 10
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
merge newbranch
```

### code
```shell
echo "HelloWorld"
```

```python
@requires_authorization
def somefunc(param1='', param2=0):
    r'''A docstring'''
    # interesting
    if param1 > param2: 
        print 'Gre\'ater'
    return (param2 - param1 + 1) or None

class SomeClass:
    pass
```

### Latex

$$ 1 + 1 = ? $$
