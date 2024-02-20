CRUD

En rails el “model” es el modelo de “Active Record”

rails g model restaurant name:string rating:integer

Hacemos eso y active record ya tiene modelos y generadores ya automaticos que definen el modelo con lo que le paso, y se crean en /db/migrate.

dentro de la carpeta “db”, en “migrate” ya tenemos nuestra clase recien creada con el nombre y los dos keys y valores que definimos en la declaracion anterior ya creados. Hacemos entonces “rails db:migrate” y creamos el schema en la carpeta “migrate”.

Cuando migramos, si la cagamos podemos poner “db: rollback” y elimina la tabla recien creada. Se puede rollback varias veces pero va a ir una a una hacia atras. Lo que hace basicamente es un “drop table”.


La consola → “rails c”

no hay mas “require relative”

Restaurant.create -> creamos una instancia de un restaurante
Se usan parentesis curvos, no hay que hacer el curly braces.

Restaurant.create(name: “Pizza place”, rating: 3, address: “Next door”)

Antes en Ruby teniamos que hacer “insert” dps de esto, pero en Rails con el .create ya está todo hecho.

Sandbox → si queres chivear haces “rails c –sandbox” y cualquier modificación que hagas a la data luego va a hacer rollback. Pero cualquier cambio que hagamos en el sandbox se va a ver reflejado en el server, dps se borra.


CRUD actions

Read(all)
GET http://localhost:300/restaurants

Se va al archivo “routes”

Rails.application.routes.draw do
    get “/restaurants”, to: “restaurants#index”
end

Reminder: para chequear las rutas que cree se va a la consola y se hace “rails routes”
claramente si vamos a la ruta que creamos va a dar un error pq tenemos que crear un controlador para “restaurants”

rails g controller RestaurantsController (siempre plural) 

Vamos al RestaurantsController y creamos la accion.

Ahora tenemos que crear el view para el “index”, cada vez que hacemos un controller vamos y creamos el view para ese controller.

En el controlador tengo que crear una instance variable: @restaurants = Restaurant.all

Eso va a decir que en esa variable vamos a guardar la info de todos los restaurantes (y la vamos a poder llamar desde los demás archivos).

En el view ahora tengo que darle forma a la info, voy a necesitar acceder a @restaurants e iterar con un “.each do |restaurant|” y dps restaurant.key (pedimos que nos pase ya sea el nombre, la direccion o el rating, todo en formato erb)
Al estar en un active record es que podemos hacer restaurant.key, sino teniamos que hacer como si fuera un hash con los [ ].

Se lo formatea tmb para que se despliegue bien en pantalla con los tags propios de html, dps les podremos lo de bootstrap y etc...

Si ahora ponemos en la url “/restaurants/3” (el id), nos va a dar un error pq si bien tenemos la ruta hecha. la tenemos que asignar al id, sino imposible.

Entonces vamos al routes.db y hacemos un get “/restaurants/:id”, to: “restaurants#show”

“show” -> la accion por defecto cuando no sabemos que poner. de todas maneras la tenemos que definir en el controller, sino de nuevo va a dar un error. y tmb el view.

Entonces definimos el controller como en el anterior solo que esta vez “@restaurant = Restaurant.find(1)” (id del restaurante), pero para leer la id tenemos que usar “params[:id]”. 
Al tratarse de un método lo podemos hacer directamente con un key, sino tendríamos que poner id como string. 

Y en el .erb del view vamos y le damos forma de nuevo utilizando la instance variable @restaurants de nuevo.

Si queremos agregar un link, tenemos que ir al index.html.erb y hacemos 
<%= link_to “See more”, restaurants_path %>. pero a esto lo tenemos que linkear al id.

Entonces en el routes agregamos el “/restaurants/:id”, to: “restaurants#show as :restaurant”restaurant_path(restaurant) (ver como se hizo dps)

en el controller entonces el show seria @restaurant = Restaurant.find (params[:id]).


Hasta entonces estabamos haciendo GET, estabamos leyendo nada mas.

Ahora tenemos que aprender a hacer POST, que es basicamente modificar.
El crear requiere de 2 acciones, show o mostrar el html del form, y el post para manejar el submit del form.
Entonces en routes.db hacemos un 
get “restaurants/new”, to: “restaurants#new”. → el new es por convencion cuando vamos a crear.

Si vamos al server y vamos a la ruta eso va a dar un error, entonces para prevernr eso tenemos que poner la linea en route como la segunda, antres de la ultima que habiamos creado.

Luego el mismo proceso, hacemos el controller, y dps el view.

en el controller entonces
def new
    @restaurant = Restaurant.new
end

En el view vamos a tener que crear el form.
action = “/restaurants”, method = “post”, etc…

Hay un metodo de rails para hacer el form para nosotros. (repasar formmmsss)

hay un metodo <%= form_with model: @restaurant do |form (o f)| %> → esto es una instancia que recorre el form

entones dps hacemos f.label :name, f.text_field :name, f.label :address, f.label :rating, f.number_field :rating, f.submit (todo dentro de los tags erb)
en f.submit podemos poner un string y eso es lo que va a decir el boton, el default va a ser “create” y el nombre.

El codigo se da cuenta que al pasar @restaurants sin un id, estamos creando uno nuevo. por que? por que en el controller new, pusimos @restaurants = Restaurant.new

Dps de hacer el form, tenemos que crear la ruta en routes, esta vez con post
post “/restaurants to: restaurants#create”

en el controller tmb tenemos que crear el controlador para create,.

def create
    restaurant = Restaurant.new(params[:restaurant])
    restaurant.save
end

Si ejecutamos esto va a dar error de vuelta, esto es por el “permitted: false” (si hacemos un raise en la consola lo vemos). Basicamente Ruby no deja guardar nada por defecto a menos que si lo especifiquemos, entonces tenemos que hacerlo.

entonces vamos al controller de vuelta, pero de manera privada (private), definimos 


def restaurant_params
    params.require(:restaurant).permit(:name, :adress, :raiting)
end

De esta manera le especificamos a rails cuales van a ser los parametros permitidos a la hora de submitir un form y podemos evitar que nos hackeen y creen cualquier cosa a piaccere.

Pero entonces en create (dentro del controller), tenemos que actualizar

restaurant = Restaurant.new(restaurant_params)
restaurant.save
redirect_to restaurant_path(restaurant) → queremos dirigirnos al restaurante que acabamos de crear


UPDATE

Consta de dos partes, un GET como el anterior, y un PATCH, para actualizar los parametros existentes.

Entonces de nuevo, en el routes.rb

get “restaurants/:id/edit”, to: “restaurants#edit”, as “edit_restaurant”
patch “restaurants/:id”, to: “restaurants#update”

En el controller defimimos el método

Si hacemos un raise, vemos que vamos a necesitar la misma ruta que hicimos en el paso anterior (show)

Entonces creamos 

def edit
    @restaurant = Restaurant.find(params[:id])    
end

Pero vamos a tener que definir un metodo update:

def update
    @restaurant = Restaurant.find(params[:id])
    restaurant.update(restaurant_params)
    redirect_to restaurant_path (:restaurant)
end

→ dps de actualizar los queremos llevar al view del restaurante que acabamos de editar


Entonces para borrar

Routes de nuevo

delete “restaurants/:id”, to: “restaurants#destroy


Entonces en el controller

def destroy
    @restaurant = Restaurant.find(params[:id])
    @restaurant.destroy
    redirect_to restaurants_path (los llevamos de vuelta al inicio dps), status: :see_other (dar informacion al usuario)
end


Entonces dps en el view

<%= link_to “Delete restaurant”, destroy_restaurant_path(@restaurant), data: {turbo_method: :delete, turbo_confirm: “ARE YOU SURE?”} %>

Entonces finalmente fue todo al pedo, en routes podemos comentar todo a la mierda y poner:

resources :restaurants


Ver que carajo hicimos en partials dps de borrar todo en el router pq no entendi.


