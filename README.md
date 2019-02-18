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

### Adding cocoon gem

- add the cocoon gem
- bundle
- update the app.js

```
//= require jquery
//= require rails-ujs
//= require bootstrap-sprockets
//= require cocoon
//= require activestorage
//= require turbolinks
//= require_tree .

```

- rails g model Ingredient name recipe:belongs_to
- rails g model Direction step:text recipe:belongs_to
- rails db:migrate
- update recipe.rb with the ingredient and direction
- inverse of is [explained here](https://www.rubydoc.info/gems/cocoon/1.2.12)

```
	has_many :ingredients, inverse_of: :recipe
	has_many :directions, inverse_of: :recipe

	accepts_nested_attributes_for :ingredients,
  															reject_if: proc { |attributes| attributes['name'].blank? },
  															allow_destroy: true
 	accepts_nested_attributes_for :directions,
  															reject_if: proc { |attributes| attributes['step'].blank? },
  															allow_destroy: true

  validates :title, :description, :image, presence: true
```

- update the params in the recipes controller

```
def recipe_params
  params.require(:recipe).permit(:title, :description, :user_id, :image,
                                  ingredients_attributes: [:id, :name, :_destroy], 
                                  directions_attributes: [:id, :step, :_destroy])
end
```

- create the partial recipes/ingredient_fields.html.haml

```
.form-inline.clearfix
	.nested-fields
		= f.input :name, input_html: { class: 'form-input form-control' }
		= link_to_remove_association "Remove", f, class: "form-button btn btn-default"
```

- create the partial recipes/direction_fields.html.haml

```
.form-inline.clearfix
	.nested-fields
		= f.input :step, input_html: { class: 'form-input form-control' }
		= link_to_remove_association "Remove Step", f, class: "btn btn-default form-button"
```

- add devise gem
- rails g devise:install
- rails g devise:views
- rails g devise User
- rails db:migrate
- update user.rb

```
has_many :recipes
```

- update recipe.rb

```
belongs_to :user
```

- update the recipes controller to link with user and request_permission to not allow other users to go do recipe/1/edit if it is not their recipe

```
class RecipesController < ApplicationController
  before_action :set_recipe, only: [:show, :edit, :update, :destroy]
  before_action :require_permission, only: [:edit, :update, :destroy]
  before_action :authenticate_user!, except: [:index, :show]
  # GET /recipes
  # GET /recipes.json
  def index
    @recipes = Recipe.all.order("created_at DESC")
  end

  # GET /recipes/1
  # GET /recipes/1.json
  def show
  end

  # GET /recipes/new
  def new
    @recipe = current_user.recipes.build
  end

  # GET /recipes/1/edit
  def edit
  end

  # POST /recipes
  # POST /recipes.json
  def create
    @recipe = current_user.recipes.build(recipe_params)

    respond_to do |format|
      if @recipe.save
        format.html { redirect_to @recipe, notice: 'Recipe was successfully created.' }
        format.json { render :show, status: :created, location: @recipe }
      else
        format.html { render :new }
        format.json { render json: @recipe.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /recipes/1
  # PATCH/PUT /recipes/1.json
  def update
    respond_to do |format|
      if @recipe.update(recipe_params)
        format.html { redirect_to @recipe, notice: 'Recipe was successfully updated.' }
        format.json { render :show, status: :ok, location: @recipe }
      else
        format.html { render :edit }
        format.json { render json: @recipe.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /recipes/1
  # DELETE /recipes/1.json
  def destroy
    @recipe.destroy
    respond_to do |format|
      format.html { redirect_to recipes_url, notice: 'Recipe was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_recipe
      @recipe = Recipe.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def recipe_params
      params.require(:recipe).permit(:title, :description, :user_id, :image,
                                      ingredients_attributes: [:id, :name, :_destroy], 
                                      directions_attributes: [:id, :step, :_destroy])
    end

  def require_permission
    @user = current_user
    if !user_signed_in? || @recipe.user_id != @user.id
      redirect_to root_path, notice: "Sorry, you're not allowed to view that page"
    end
  end
end
```