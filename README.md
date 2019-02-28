# evote-movie-2019-14-edit-form

Part of the progressive Movie Voting website project at:
https://github.com/dr-matt-smith/evote-movie-2019

The project has been refactored as follows:

- create a template file to display a form for editing Movies `/templates/editMovieForm.php`:

    ```php
      <?php
      require_once __DIR__ . '/_header.php';
      ?>
      
      <h1>EDIT New Movie</h1>
      
      <form
              action="index.php"
              method="GET"
      >
      
          <input type="hidden" name="id" value="<?= $movie->getId() ?>">
      
          <input type="hidden" name="action" value="processUpdateMovie">
      
          Title:
          <input name="title" value="<?= $movie->getTitle() ?>">
      
          <br>
          Price:
          <input name="price"  value="<?= $movie->getPrice() ?>">
      
          <br>
          <input type="submit">
      </form>
      
      
      <?php
      require_once __DIR__ . '/_footer.php';
      ?>
    ```
      
- add to the Front Controller `/public/index.php` a case where `action=editMovieForm` invokes the `editForm()` method of an `AdminController` object:

    ```php
        // ------ admin section --------
        case 'newMovieForm':
            $adminController->newMovieForm();
            break;
    
        case 'createNewMovie':
            $adminController->createNewMovie();
            break;
    
        case 'deleteMovie':
            $adminController->deleteMovie();
            break;
    
        case 'editMovie':
            $id = filter_input(INPUT_GET, 'id');
            $adminController->editMovieForm($id);
            break;
    ```
    
    - NOTE: in this example, the `id` value has been extracted from the `GET` Request, and is passed directly to the method `editMovieForm($id)`
    
- add method editMovieForm($id)` to the `AdminController` class in `/src/AdminController.php`:

    ```php
          public function editMovieForm($id)
          {
              $movieRepository = new MovieRepository();
              $movie = $this->movieRepository->getOneById($id);
      
              $pageTitle = 'edit movie';
              require_once __DIR__ . '/../templates/editMovieForm.php';
          }
    ```
    
- update the movie list template `/templates/list.php` to add an `EDIT` link in an extra column, passing the `id` along with the action `editMovie` to the Front Controller:

    ```php
        <?php
            foreach($movies as $movie):
        ?>        
            <tr>
                <td><?= $movie->getId() ?></td>
                <td><?= $movie->getTitle() ?></td>
                <td>&euro; <?= $movie->getPrice() ?></td>

                <td>
                    <a href="index.php?action=deleteMovie&id=<?= $movie->getId() ?>">DELETE</a>
                </td>

                <td>
                    <a href="index.php?action=editMovie&id=<?= $movie->getId() ?>">EDIT</a>
                </td>
            </tr>
    ```
    
- create an `AdminController` method to process submitted form data about a changed movie:

    ```php
          public function processUpdateMovie()
          {
              $id= filter_input(INPUT_GET, 'id');
              $title = filter_input(INPUT_GET, 'title');
              $price = filter_input(INPUT_GET, 'price');
      
              $movie = new Movie();
              $movie->setId($id);
              $movie->setTitle($title);
              $movie->setPrice($price);
      
              $movieRepository = new MovieRepository();
              $success = $movieRepository->update($movie);
      
      
              if($success){
                  $mainController = new MainController();
                  $mainController->listMovies();
              } else {
                  $errors[] = "error trying to UPDATE with id = '$id', title = '$title', price = '$price'";
                  require_once __DIR__ . '/../templates/error.php';
              }
          }
    ```
    
      
- add to the Front Controller `/public/index.php` a case where `action=processUpdateMovie` invokes the `processUpdateMovie()` method of an `AdminController` object:
    
    ```php
        case 'processUpdateMovie':
            $adminController->processUpdateMovie();
            break;
    ```

- 