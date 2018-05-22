# Taxonomía Personalizada

* [Custom Post Types](#custom-post-types)
* [Custom Taxonomies](#custom-taxonomies)
* [Custom Fields](#custom-fields)
* [Metaboxes](#metaboxes)

## Custom Post Types

Un post es cada uno de los contenidos que WordPress puede guardar.

Los post predeterminados son:

* Entradas (post)
* Páginas (page)
* Archivos adjuntos (attachment)
* Revisiones (revision)
* Menús (nav_menu_item)

Los custom pos types pueden actuar como entradas o páginas y se pueden organizar en categorías, etiquetas o taxonomías.

###  Entradas vs Páginas

Entradas | Páginas
-- | --
Orden cronológico | Es sólo un elemento
No tienen padres o hijos | Puede tener padre o hijos
Tiene categorías | No tiene categorías
Tiene etiquetas | No tiene etiquetas

### Más info

* [Developers WordPress](https://developer.wordpress.org/themes/basics/post-types/)
* [Codex WordPress](https://codex.wordpress.org/Post_Types)
* [Generate WP](https://generatewp.com/post-type/)
* [Custom Post Type UI](https://es.wordpress.org/plugins/custom-post-type-ui/)

```php
<?
function custom_post_type() {
  $labels = array(
    'name' => _x( 'Post Types', 'Post Type General Name', 'mawt' ),
    'singular_name' => _x( 'Post Type', 'Post Type Singular Name', 'mawt' ),
    'menu_name' => __( 'Post Types', 'mawt' ),
    'name_admin_bar' => __( 'Post Type', 'mawt' ),
    'archives' => __( 'Item Archives', 'mawt' ),
    'attributes' => __( 'Item Attributes', 'mawt' ),
    'parent_item_colon' => __( 'Parent Item:', 'mawt' ),
    'all_items' => __( 'All Items', 'mawt' ),
    'add_new_item' => __( 'Add New Item', 'mawt' ),
    'add_new' => __( 'Add New', 'mawt' ),
    'new_item' => __( 'New Item', 'mawt' ),
    'edit_item' => __( 'Edit Item', 'mawt' ),
    'update_item' => __( 'Update Item', 'mawt' ),
    'view_item' => __( 'View Item', 'mawt' ),
    'view_items' => __( 'View Items', 'mawt' ),
    'search_items' => __( 'Search Item', 'mawt' ),
    'not_found' => __( 'Not found', 'mawt' ),
    'not_found_in_trash' => __( 'Not found in Trash', 'mawt' ),
    'featured_image' => __( 'Featured Image', 'mawt' ),
    'set_featured_image' => __( 'Set featured image', 'mawt' ),
    'remove_featured_image' => __( 'Remove featured image', 'mawt' ),
    'use_featured_image' => __( 'Use as featured image', 'mawt' ),
    'insert_into_item' => __( 'Insert into item', 'mawt' ),
    'uploaded_to_this_item' => __( 'Uploaded to this item', 'mawt' ),
    'items_list' => __( 'Items list', 'mawt' ),
    'items_list_navigation' => __( 'Items list navigation', 'mawt' ),
    'filter_items_list' => __( 'Filter items list', 'mawt' ),
  );

  $args = array(
    'label' => __( 'Post Type', 'mawt' ),
    'description' => __( 'Post Type Description', 'mawt' ),
    'labels' => $labels,
    // las taxonomías que soportará
    'taxonomies' => array( 'category', 'post_tag' ),
    // Todo lo que soporta este post type
		'supports' => array( 'title', 'editor', 'excerpt', 'author', 'thumbnail', 'comments', 'revisions', 'custom-fields' ),
		//hierarchical true se comporta como página, false como entrada
		'hierarchical' => false,
    'public' => true,
    'show_ui' => true,
    'show_in_menu' => true,
    'menu_position' => 5,
    //El icono que tendrá https://developer.wordpress.org/resource/dashicons
    'menu_icon' => 'dashicons-welcome-view-site',
    'show_in_admin_bar' => true,
    'show_in_nav_menus' => true,
    'can_export' => true,
    'has_archive' => true,
    'exclude_from_search' => false,
    'publicly_queryable' => true,
    'capability_type' => 'page',
  );

  register_post_type( 'a_post_type', $args );
}

add_action( 'init', 'custom_post_type', 0 );
```

Todos los custom post type que realicemos __no se mostrarán en el loop__ por defecto. Para que el loop lo muestre debemos de 
realizar una pequeña modificación.

1. Nos dirigimos a nuestro archivo __functions.php__ y cargamos un nuevo módulo que vamos a desarrollar. Le podemos dar el siguiente nombre;

````php
<?
//...
require_once get_template_directory() . '/inc/custom-pre-get-posts.php';
//...
````

2. En ``./inc/`` creamos nuestro arhivo ``cusmtom-pre-get-post.php`` y añadimos el siguiente código

````php
<?
if ( !function_exists( 'mawt_show_post_types_in_loop' ) ):
  function mawt_show_post_types_in_loop ( $query ) {
    // que no sea el admin y sea el query principal
    if ( !is_admin() && $query->is_main_query() ):
      $query->set( 'post_type', array( 'post', 'page', 'cuidados' ) );
    endif;
  }
endif;

add_action( 'pre_get_posts', 'mawt_show_post_types_in_loop' );
````
Cada vez que agreguemos nuevos tipos de post tenemos que actualizar nuestro módulo para irle agregando los nuevos custom post type que vayamos creando

`$query->set( 'post_type', array( 'post', 'page', 'cuidados','otrocusmpost','otromas','...' ) );`

3. En el __content.php__ (nuestro contenido principal en donde usamos el loop) tenemos que usar la función `wp_reset_query();` que lo que hace es limpiar la variable de la consulta. Evitamos provblemas de visualización.
La función __wp_reset_query();__ se usa cuando usemos WP-Query. Pero es muy buena práctica usarla tb en donde hagamos loop principal (content.php).



**[⬆ regresar al índice](#taxonomía-personalizada)**

## Custom Taxonomies

Las taxonomías son la forma de agrupar contenido en WordPress, permiten relacionar contenido mediante términos en común.

Las taxonomías predeterminadas son:

* Categorías (category)
* Etiquetas (tag)

Las custom taxonomies pueden actuar como categorías o etiquetas.

###  Categorías vs Etiquetas

Categorías | Etiquetas
-- | --
Son obligatorias, toda entrada tiene al menos una | Son opcionales
Son jerárquicas (padres o hijos) | No son jerárquicas
Primer método para agrupar contenido en WordPress | Segundo método para agrupar contenido en WordPress

### Más info

* [Developers WordPress](https://developer.wordpress.org/themes/basics/categories-tags-custom-taxonomies/)
* [Codex WordPress](https://codex.wordpress.org/Taxonomies)
* [Generate WP](https://generatewp.com/taxonomy/)
* [Custom Post Type UI](https://es.wordpress.org/plugins/custom-post-type-ui/)

```php
<?
function custom_taxonomy() {
  $labels = array(
    'name' => _x( 'Taxonomies', 'Taxonomy General Name', 'mawt' ),
    'singular_name' => _x( 'Taxonomy', 'Taxonomy Singular Name', 'mawt' ),
    'menu_name' => __( 'Taxonomy', 'mawt' ),
    'all_items' => __( 'All Items', 'mawt' ),
    'parent_item' => __( 'Parent Item', 'mawt' ),
    'parent_item_colon' => __( 'Parent Item:', 'mawt' ),
    'new_item_name' => __( 'New Item Name', 'mawt' ),
    'add_new_item' => __( 'Add New Item', 'mawt' ),
    'edit_item' => __( 'Edit Item', 'mawt' ),
    'update_item' => __( 'Update Item', 'mawt' ),
    'view_item' => __( 'View Item', 'mawt' ),
    'separate_items_with_commas' => __( 'Separate items with commas', 'mawt' ),
    'add_or_remove_items' => __( 'Add or remove items', 'mawt' ),
    'choose_from_most_used' => __( 'Choose from the most used', 'mawt' ),
    'popular_items' => __( 'Popular Items', 'mawt' ),
    'search_items' => __( 'Search Items', 'mawt' ),
    'not_found' => __( 'Not Found', 'mawt' ),
    'no_terms' => __( 'No items', 'mawt' ),
    'items_list' => __( 'Items list', 'mawt' ),
    'items_list_navigation' => __( 'Items list navigation', 'mawt' ),
  );

  $args = array(
    'labels' => $labels,
    //hierarchical true se comporta como categoría, false como etiqueta
    'hierarchical' => false,
    'public' => true,
    'show_ui' => true,
    'show_admin_column' => true,
    'show_in_nav_menus' => true,
    'show_tagcloud' => true,
  );
  register_taxonomy( 'a_taxonomy', array( 'a_post_type' ), $args );
}

add_action( 'init', 'custom_taxonomy', 0 );
```

**[⬆ regresar al índice](#taxonomía-personalizada)**

## Custom Fields

Los campos personalizados pueden utilizarse para añadir metadatos extra a un post, son nativos de WordPress.

Los metadatos se manejan en pares de clave/valor (Key/Value). La clave es el nombre del metadato, mientras que el valor es la información que aparecerá en la lista de metadatos de cada entrada individual a la que la información esté asociada.

### Más info

* [Codex WordPress](https://codex.wordpress.org/es:Using_Custom_Fields)
* [Advanced Custom Fields](https://wordpress.org/plugins/advanced-custom-fields/)
* [CMB2](https://wordpress.org/plugins/cmb2/)


Para mostrar los metadatos tenemos que poner en nuestro loop principal este código (a base de ejemplo práctico); 

````html
 <div>
            <h3>Custom Fields & Metaboxes</h3>
            <?php the_meta(); ?>
            <p>
            <!-- Custom Fields -->
              <?php echo get_post_meta( get_the_ID(), 'origen', true); ?>
            </p>
            <p>
              `
            </p>
            <p>
            <!-- Metaboxes -->
              <?php echo get_post_meta( get_the_ID(), 'mb_origen_raza', true); ?>
            </p>
            <p>
              <?php echo get_post_meta( get_the_ID(), 'mb_esperanza_vida', true); ?>
            </p>
            <h3>ACF</h3>
            <p>
              <?php the_field('ideal_para'); ?>
             </p>
            <p>
              <?php echo get_field('ideal_para'); ?>
             </p>
          </div>
````

`<?php the_meta(); ?>` Se trae todos los metadatos que haymos incorporados por medio de los __custom field__. Si queremos prescindir de el par llave/valor tendremos que indicarlo de la siguiente manera `<?php echo get_post_meta( get_the_ID(), 'mb_origen_raza', true); ?>`
Así, solo mostrará el valor.

```php
<?
the_meta();
echo get_post_meta( get_the_ID(), 'a_key', true );
```

**[⬆ regresar al índice](#taxonomía-personalizada)**

## Metaboxes

Las meta cajas permiten agregar cualquier tipo de contenido extra a los posts, texto, multimedia, embeds, etc.

A diferencia de los campos personalizados no son nativos de WordPress por lo que hay que habilitarlos.

### Más info

* [Codex WordPress](https://codex.wordpress.org/Function_Reference/do_meta_boxes)
* [Developers WordPress](https://developer.wordpress.org/reference/functions/add_meta_box/)
* [Advanced Custom Fields](https://wordpress.org/plugins/advanced-custom-fields/)
* [CMB2](https://wordpress.org/plugins/cmb2/)

Si queremos que nuestros metaboxes se muestren en paginas y post debemos indicarlo por medio de un arreglo

```php
<?
  add_meta_box( 'metabox-id', __('Metabox Title', 'mawt'), 'callback_metaboxes', array('post','page'), 'normal', 'high', null );

```

```php
<?
function add_a_metaboxes () {
  //7 parametros:
    // id para identificar el metabox
    // Titulo del Metabox
    // Callback con el contenido
    // Pantalla donde se mostrará
    // Contexto donde se mostrará (normal, aside, advanced)
    // Prioridad en la que se mostrará
    // Argumentos con callback
    
   

  add_meta_box( 'metabox-id', __('Metabox Title', 'mawt'), 'callback_metaboxes', 'post', 'normal', 'high', null );
}

function callback_metaboxes () {
  echo 'Content of Metaboxes';
}

add_action('add_meta_boxes', 'add_a_metaboxes');
```

En WP-Generator metabox es de pago.  El uso de metabox requiere consultar la BBDD de la aplicación y ello conyeva mucho trabajo. 

`wp_nonce_field( basename(__FILE__), 'mawt_nonce' );` Es una función de WP para crear un campo oculto que vamos a usar con posterioridad. __wp_nonce_field__ son funciones que generan campos de control cada 24h pero es un tema más de desarrollo de plugins. 
Luego observamos que tenemos una tabla HTML para recoger la información. Es decir, tenemos que contruir nuestro formulario para recoger la información y luego
también debemos de ejecutar una función que este pendiente que cuando salvamos nuestra publicación, recoga y guarde toda la información.

`add_action('save_post', 'mawt_save_metaboxes', 10, 3);`

Cuando salvamos la publicación esta función `mawt_save_metaboxes`  nos permite guardar toda la información recogida.

` update_post_meta($post_id, 'mb_esperanza_vida', $mb_esperanza_vida );` Actualiza nuestra información.
```php
<?
if ( !function_exists('mawt_metaboxes') ):
  function mawt_metaboxes ( $post ) {
    wp_nonce_field( basename(__FILE__), 'mawt_nonce' );
?>
    <table class="form-table">
      <tr>
          <th class="row-title" colspan="2">
              <p>Añade la información de la raza</p>
          </th>
      </tr>
      <tr>
          <th class="row-title">
              <label for="mb_origen_raza">Origen de la Raza:</label>
          </th>
          <td>
              <input id="mb_origen_raza" name="mb_origen_raza" class="regular-text" type="text" value="<?php echo esc_attr( get_post_meta( $post->ID, 'mb_origen_raza', true ) ); ?>">
          </td>
      </tr>
      <tr>
          <th class="row-title">
              <label for="mb_esperanza_vida">Esperanza de Vida:</label>
          </th>
          <td>
            <?php $res_esperanza = esc_html( get_post_meta( $post->ID, 'mb_esperanza_vida', true ) );  ?>
            <select name="mb_esperanza_vida" id="mb_esperanza_vida" class="post-box">
              <option value="" selected>- - -</option>
              <option <?php selected($res_esperanza, '8-10'); ?> value="8 a 10 años">8 a 10 años</option>
              <option <?php selected($res_esperanza, '10-12'); ?> value="10 a 12 años">10 a 12 años</option>
              <option <?php selected($res_esperanza, '12-14'); ?> value="12 a 14 años">12 a 14 años</option>
              <option <?php selected($res_esperanza, '15-20'); ?> value="15 a 20 años">15 a 20 años</option>
            </select>
          </td>
      </tr>
    </table>
<?php
  }
endif;

if ( !function_exists( 'mawt_save_metaboxes' ) ):
  function mawt_save_metaboxes( $post_id, $post, $update ) {
    if ( !isset( $_POST[ 'mawt_nonce' ] ) || !wp_verify_nonce( $_POST[ 'mawt_nonce' ], basename( __FILE__ ) ) )
      return $post_id;

    if ( !current_user_can( 'edit_post', $post_id ) )
      return $post_id;

    if( defined( 'DOING_AUTOSAVE' ) && DOING_AUTOSAVE )
      return $post_id;

    $mb_origen_raza = $mb_esperanza_vida = '';

    if( isset( $_POST['mb_origen_raza'] ) ) {
      $mb_origen_raza = sanitize_text_field( $_POST['mb_origen_raza'] );
    }

    update_post_meta($post_id, 'mb_origen_raza', $mb_origen_raza );

    if( isset( $_POST['mb_esperanza_vida'] ) ) {
      $mb_esperanza_vida = sanitize_text_field( $_POST['mb_esperanza_vida'] );
    }

    update_post_meta($post_id, 'mb_esperanza_vida', $mb_esperanza_vida );
  }
endif;

add_action('save_post', 'mawt_save_metaboxes', 10, 3);
?>
```

Para imprimir los metaboxes en content.php

````html
 <!-- Metaboxes -->
              <?php echo get_post_meta( get_the_ID(), 'mb_origen_raza', true); ?>
            </p>
            <p>
              <?php echo get_post_meta( get_the_ID(), 'mb_esperanza_vida', true); ?>
            </p>
<!-- ... -->
````
__Los metaboxes si cambiamos de tema se pierden, en cambio, los cusmtom field se mantienen aunque cambiemos de tema__

**[⬆ regresar al índice](#taxonomía-personalizada)**