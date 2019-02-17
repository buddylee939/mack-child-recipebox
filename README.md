# Steps

- rails new recipebox
- installing haml gem
- install bootstrap sass
- install jquery rails
- install simple form
- bundle
- change app.scss
- rails generate simple_form:install --bootstrap
- rails g scaffold Recipe title description:text user_id:integer {he did the controller/model way}
- rails db:migrate
- routes: root 'recipes#index'
- delete scaffold scss
- he added paperclip
- rails active_storage:install
- rails db:migrate
- add to recipe.rb model

```
has_one_attached :image
```

- update recipe form partial

```
<%= f.input :image %>
```

- update the recipes controller recipes params to include the image

```
def recipe_params
  params.require(:recipe).permit(:title, :description, :user_id, :image)
end
```

- update the recipes show page to show the image

```
<p>
	<%= image_tag @recipe.image %>
</p>
```

- 