# Яковлев Эрик Денисович БПИ239
## 1. Модификация алгоритма Дейкстры

При поиске кратчайшего пути в произведения весов вместо суммы нужно заменить сложение на умножение, а сравнение на сравнение произведений.
То есть пусть для каждой вершины v мы будем хранить стоимость пути d[v], равную произведению весов ребер вдоль пути от стартовой вершины s
(при этом d[s] = 1). При релаксации ребра (u,v) выполняется условие:  
если  d[v] > d[u] * w(u,v), то d[v] = d[u] * w(u,v).

В стандартном алгоритме сумма не уменьшается при добавлении неотрицательных слагаемых. Аналогично, чтобы в произведении при добавлении очередного ребра стоимость не уменьшалась
необходимо, чтобы каждый множитель удовлетворял условию w(u,v) ≥ 1. Тогда при переходе от u к v произведение не уменьшается, и жадная стратегия Дейкстры остаётся корректной.

**Вывод:** Модифицированный алгоритм DijkstraMULT(G,s) корректно находит пути в графах с минимальной стоимостью
у которых все веса удовлетворяют условию w(u,v) ≥ 1. Если же встречаются ребра с весами, меньшими единицы (например, w(u,v) in (0,1)),
то при умножении стоимость может уменьшаться, и базовая предпосылка алгоритма (фиксированность найденного оптимума при извлечении вершины) нарушается.

*Пример подтверждения:*  
– Пусть G имеет ребра: (s -> a) с весом 2, (a -> b) с весом 3 и (s -> b) с весом 7. Тогда путь (s -> a -> b) имеет стоимость (2 * 3=6),
что меньше прямой стоимости 7. Алгоритм с операцией умножения (при (w ≥ 1)) корректно найдёт путь (s -> a -> b).

## 2. Алгоритм восстановления графа

**Идея:** Матрица кратчайших расстояний удовлетворяет условию, что для любых трёх вершин i, j, и k: dist[i][j] ≤ dist[i][k] + dist[k][j],
поэтому можно восстановить граф, оставив только те ребра, которые действительно нужны, а лишние опустить.

**Псевдокод:**

```
function ResreGraph(dist[][]):
    for each i ∈ V:
        for each j ∈ V, j > i:
            flag = true
            for each k ∈ V, k ≠ i and k ≠ j:
                if dist[i][j] == dist[i][k] + dist[k][j]:
                    flag = false
                    break
            if flag:
                // ребро (i, j) неразложимое
                AddEdge(i, j, weight = dist[i][j])
    return Graph 
```

То есть мы рассматриваем пару вершин (i,j) и проверяем, можно ли разложить расстояние dist[i][j] как сумму расстояний через какую-либо другую вершину k.
Если нет, то делаем вывод, что между i и j должно быть прямое ребро с весом dist[i][j].

**Случаи невозможности восстановления:**  
– Если матрица не удовлетворяет основным свойствам, то она не может быть матрицей кратчайших путей графа.  
– Если исходный граф содержал ребра, вес которых равен сумме весов альтернативного пути,
то восстановление будет неоднозначным. В этом случае можно восстановить минимальное представление графа,
но исходное представление может быть не уникальным.

*Пример:*  
– Для дерева матрица кратчайших расстояний однозначно определяет ребра, и алгоритм восстановит именно дерево.  
– Для полноcвязного графа с несколькими кратчайшими путями выбор прямых ребер может быть неоднозначным, но минимальное представление восстановить можно.

## 3. Ошибка в реализации алгоритма

Код из задания:
```
for k = 1 to n:
    for i = 1 to n:
        for j = 1 to n:
            dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```

В приведённой реализации порядок циклов нарушен:
```
for i = 1 to n:
    for j = 1 to n:
        for k = 1 to n:
            dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])
```
При правильном алгоритме внешний цикл по k гарантирует, что когда мы пытаемся улучшить dist[i][j] с помощью вершины k, уже учтены все кратчайшие пути,
использующие промежуточные вершины с номерами меньше k. При неверном порядке (сначала по i и j) эта индуктивная гипотеза нарушается:
для фиксированной пары (i,j) рассматриваются все k одновременно, и обновлённое значение dist[i][j] может основываться на незавершённых вычислениях,
когда необходимые улучшения через некоторые промежуточные вершины ещё не были гарантированно получены.

**Контрпример:**  
Рассмотрим ориентированный граф с четырьмя вершинами и следующими ребрами (веса – неотрицательные):

- (1 -> 2) с весом 3  
- (1 -> 3) с весом 8  
- (2 -> 3) с весом 2  
- (2 -> 4) с весом 10  
- (3 -> 4) с весом 1  
- (1 ->4) с весом 10

**Ожидаемые кратчайшие расстояния:**  
– (dist[1][3] = min8, 3+2=5 = 5)  
– (dist[1][4] = min10, 3+10=13, 5+1=6 = 6)

При корректном порядке (сначала по k) алгоритм гарантированно найдёт эти значения.
При же неправильном порядке (сначала i и j) может оказаться, что порядок обновлений таков, что некоторые улучшения (например, обновление dist[1][3] до 5)
не используются при дальнейших релаксациях, и итоговое значение dist[1][4] может остаться завышенным (например, равным 10).
Без строгой индуктивной структуры алгоритм может пропустить комбинацию путей, где сначала требуется обновить dist[1][3],
а затем уже использовать его для улучшения dist[1][4].

**Вывод:**  
Чтобы обеспечить корректность, внешний цикл должен идти по k. Иначе может получиться ситуация, когда оптимальный путь, использующий цепочку промежуточных вершин, не будет найден.

## 4. Наличие общего ребра в кратчайших путях

Да, такое возможно. Для этого необходимо, чтобы a и b принадлежали одному циклу, а оптимальные пути между ними заставляют использовать общий сегмент (v<sub>i</sub>,v<sub>i</sub>).

**Характеристика такого графа:**

– Обязательно должно быть без отрицательных циклов, чтобы кратчайшие пути были корректно определены.  

– Пусть оптимальный путь из a в b имеет вид  

a -> ... -> v<sub>i</sub> -> v<sub>i</sub> -> ... -> b,

а кратчайший путь из b в a имеет вид  

b -> ... -> v<sub>i</sub> -> v<sub>i</sub> -> ... -> a.

Это означает, что ребро (v<sub>i</sub>,v<sub>i</sub>) является общим звеном в обоих направлениях. Такое может произойти, если наиболее выгодный способ
соединить некоторые части цикла – именно через это ребро.

**Пример:**  
Рассмотрим граф с вершинами (a, v<sub>i</sub>, v<sub>i</sub>, b) и следующими ребрами:
- (a -> v<sub>i</sub>) с весом 1  
- (v<sub>i</sub> -> v<sub>i</sub>) с весом 1  
- (v<sub>i</sub> -> b) с весом 1  
- (b -> v<sub>i</sub>) с весом 100  
- (b -> a) с весом 5  
- (v<sub>i</sub> -> a) с весом 100  

Здесь кратчайший путь от a к b – это (a -> v<sub>i</sub> -> v<sub>i</sub> -> b) (общий вес 3). Если дополнительно добавить ребра так, чтобы путь от b к a вынужденно использовал цепочку
(v<sub>i</sub> -> v<sub>i</sub>) (например, сделать альтернативный прямой путь менее выгодным), то можно получить ситуацию, когда и путь (b -> ... -> v<sub>i</sub> -> v<sub>i</sub> -> ...  -> a) оптимален.

**Ограничения алгоритмов:**  
– Алгоритмы поиска кратчайших путей работают корректно, если отсутствуют отрицательные циклы.  
– Наличие общего ребра в кратчайших путях в обоих направлениях само по себе не создаёт дополнительных ограничений - оно является следствием топологии графа и структуры весов.  
– Если в графе существуют альтернативные кратчайшие пути, то восстановление или интерпретация промежуточных вершин может быть неоднозначной.
