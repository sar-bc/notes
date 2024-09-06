Файл views.py 
Классы представлений
=====================================
```python
from django.views import View

class AddPage(View):
    def get(self, request):
        form = AddPostForm()
        return render(request, 'women/addpage.html', {'menu': menu, 'title': 'Добавление статьи', 'form': form})
 
    def post(self, request):
        form = AddPostForm(request.POST, request.FILES)
        if form.is_valid():
            form.save()
            return redirect('home')
 
        return render(request, 'women/addpage.html', {'menu': menu, 'title': 'Добавление статьи', 'form': form})
		
		
urlpatterns = [
    path('', views.index, name='home'),
    path('about/', views.about, name='about'),
    path('addpage/', views.AddPage.as_view(), name='add_page'),
    ...
]		
```
В принципе, с помощью метода get_context_data() можно передавать и статические и динамические данные, то есть, любую информацию в шаблон. Поэтому нередко можно встретить использование этого метода. 
		
```python		
def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Главная страница'
        context['menu'] = menu
        context['posts'] = Women.published.all().select_related('cat')
        context['cat_selected'] = int(self.request.GET.get('cat_id', 0))
        return context		
```
### Класс ListView
```python
from django.views.generic import ListView

class WomenHome(ListView):
    model = Women
    template_name = 'women/index.html'
    context_object_name = 'posts'
 
    def get_context_data(self, *, object_list=None, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = 'Главная страница'
        context['menu'] = menu
        context['cat_selected'] = 0
        return context
```
### Создаем класс представлений для категорий
Итак, мы с вами создали класс представления для главной страницы. Давайте повторим этот процесс и пропишем аналогичный класс для отдельных категорий. Делается это очень просто. Сначала объявим класс WomenCategory с тем же базовым классом ListView, указав те же самые атрибуты и методы:

```python
class WomenCategory(ListView):
    template_name = 'women/index.html'
    context_object_name = 'posts'
 
    def get_context_data(self, *, object_list=None, **kwargs):
        context = super().get_context_data(**kwargs)
        cat = context['posts'][0].cat
        context['title'] = 'Категория - ' + cat.name
        context['menu'] = menu
        context['cat_selected'] = cat.id
        return context 
 
    def get_queryset(self):
        return Women.published.filter(cat__slug=self.kwargs['cat_slug']).select_related('cat')
```
# Класс DetailView

```python
class ShowPost(DetailView):
    model = Women
    template_name = 'women/post.html'
	
	def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['title'] = context['post']
        context['menu'] = menu
        return context
	
	def get_object(self, queryset=None):
        return get_object_or_404(Women.published, slug=self.kwargs[self.slug_url_kwarg])
```
в url.py
```python
path('post/<slug:post_slug>/', views.ShowPost.as_view(), name='post'),
```
# Класс FormView
Он предназначен для автоматизации отображения и обработки HTML-форм.

```python
from django.views.generic.edit import FormView


class AddPage(FormView):
    form_class = AddPostForm
    template_name = 'women/addpage.html'
    success_url = reverse_lazy('home')
    extra_context = {
        'menu': menu,
        'title': 'Добавление статьи',
    }
	
	def form_valid(self, form):
        form.save()
        return super().form_valid(form)
```
# Классы CreateView и UpdateView
Если мы посмотрим на список классов представлений:

[Django Doc](https://docs.djangoproject.com/en/4.2/ref/class-based-views/)

то увидим, что после FormView следуют классы CreateView, UpdateView и DeleteView. По их названиям легко догадаться, что они еще более специализированы, чем класс FormView. В частности, для добавления (то есть, создания) новой статьи мы могли бы вместо FormView использовать класс CreateView. Давайте это сделаем.

Уберем из класса AddPage метод form_valid и унаследуем класс от CreateView, получим:
```python
class AddPage(CreateView):
    form_class = AddPostForm
    template_name = 'women/addpage.html'
    success_url = reverse_lazy('home')
    extra_context = {
        'menu': menu,
        'title': 'Добавление статьи',
    }
```
# Класс UpdateView
Следующий класс, который мы рассмотрим, это UpdateView. Он служит для изменения существующих записей. На данный момент у нас с вами такого функционала нет. Поэтому давайте его добавим.

Объявим класс UpdatePage (после класса AddPage) и унаследуем его от базового класса UpdateView:
```python
class UpdatePage(UpdateView):
    model = Women
    fields = ['title', 'content', 'photo', 'is_published', 'cat']
    template_name = 'women/addpage.html'
    success_url = reverse_lazy('home')
    extra_context = {
        'menu': menu,
        'title': 'Редактирование статьи',
    }
	
urlpatterns = [
    ...
    path('edit/<int:pk>/', views.UpdatePage.as_view(), name='edit_page'),
    ...
]	
```

# Mixins как способ улучшения программного кода
```python
class WomenHome(DataMixin, ListView):
...


menu = [{'title': "О сайте", 'url_name': 'about'},
        {'title': "Добавить статью", 'url_name': 'add_page'},
        {'title': "Обратная связь", 'url_name': 'contact'},
        {'title': "Войти", 'url_name': 'login'}
]
 
 
class DataMixin:
    def get_mixin_context(self, context, **kwargs):
        context['menu'] = menu
        context['cat_selected'] = None
        context.update(kwargs)
        return context
```

По аналогии, меняем и все остальные классы представлений, где используется вызов get_context_data():

```python
class ShowPost(DataMixin, DetailView):
...
 
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        return self.get_mixin_context(context, title=context['post'])
 
 
class WomenCategory(DataMixin, ListView):
...
 
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        cat = context['posts'][0].cat
        return self.get_mixin_context(context,
                                      title='Категория - ' + cat.name,
                                      cat_selected=cat.id,
                                      )
 
 
class TagPostList(DataMixin, ListView):
...
 
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        tag = TagsPost.objects.get(slug=self.kwargs['tag_slug'])
        return self.get_mixin_context(context, title='Тег: ' + tag.tag)
```

Можно пойти еще дальше и оптимизировать с помощью этого же класса DataMixin определение словаря extra_context. Для этого в классе DataMixin мы вначале пропишем следующие строчки:

```python
class DataMixin:
    title_page = None
    extra_context = {}
 
    def __init__(self):
        if self.title_page:
            self.extra_context['title'] = self.title_page
 
        if 'menu' not in self.extra_context:
            self.extra_context['menu'] = menu
 
    def get_mixin_context(self, context, **kwargs):
        if self.title_page:
            context['title'] = self.title_page
 
        context['menu'] = menu
        context['cat_selected'] = None
        context.update(kwargs)
        return context
```

Давайте внесем аналогичные изменения в другие классы, которые используют словарь extra_context:
```python
class UpdatePage(DataMixin, UpdateView):
...
    title_page = 'Редактирование статьи'
```
# Пагинация с классом ListView
Согласно документации все, что нам нужно, это определить атрибут:

paginate_by = N

указав число N – количество элементов на одной странице. Например, если в классе WomenHome прописать этот атрибут:
```python
class WomenHome(DataMixin, ListView):
    template_name = 'women/index.html'
    context_object_name = 'posts'
    paginate_by = 3
```
Затем, в шаблоне index.html пропишем содержимое этого блока следующим образом:
```html
<nav class="list-pages">
    <ul>
        {% for p in paginator.page_range %}
        <li class="page-num">
            <a href="?page={{ p }}">{{ p }}</a>
        </li>
        {% endfor %}
    </ul>
</nav>
```
Однако это будет дублирование кода и нам проще эту строчку записать в общем классе DataMixin:
```python
class DataMixin:
    paginate_by = 3
...
```
Этот диапазон, на уровне шаблонов, можно определить через фильтр add, уменьшая значение на 2 и увеличивая на 2 относительно текущего номера страницы. Поэтому в цикле, при отображении этих номеров, мы можем записать следующее условие вывода:
```python
{% for p in paginator.page_range %}
                   {% if page_obj.number == p %}
        <li class="page-num page-num-selected">{{ p }}</li>
                   {% elif p >= page_obj.number|add:-2 and p <= page_obj.number|add:2  %}
        <li class="page-num">
            <a href="?page={{ p }}">{{ p }}</a>
        </li>
                   {% endif %}
        {% endfor %}
```
Следующим нашим улучшением будет отображение ссылок для перехода на предыдущую и следующую страницы. Это делается очень просто. В шаблоне index.html до цикла запишем такие строчки:
```python
{% if page_obj.has_previous %}
<li class="page-num">
         <a href="?page={{ page_obj.previous_page_number }}">&lt;</a>
</li>
{% endif %}
```
То же самое делаем и для отображения ссылки на следующую страницу. Только теперь добавляем пункт после цикла for:
```python
{% if page_obj.has_next %}
<li class="page-num">
         <a href="?page={{ page_obj.next_page_number }}">&gt;</a>
</li>
{% endif %}
```