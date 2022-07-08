- - -
Title: Optional features Date: 2021-08-02 Order: 5
- - -

Some features of BookWyrm have to be enabled to work.

## Geração de pré-visualização

By default, BookWyrm uses the instance's logo (or the default logo) as an OpenGraph preview image. As an alternative, you can enable the generation of preview images for books, users, and the website.

The preview images will be sized for large OpenGraph images (used by Twitter under the name of `summary_large_image`). Depending on the type of image, the contents will be:

- a imagem padrão da instância mostrará a logo grande e o nome da instância e seu endereço
- a imagem do usuário mostrará seu avatar, nome de exibição e arroba (na forma usuário@instância)
- a imagem do livro mostrará sua capa, título, subtítulo (se tiver), autor e avaliação (se tiver)

These images will be updated at various points:

- imagem da instância: quando o nome da instância ou a logo são alterados
- imagem do usuário: quando o nome de exibição ou o avatar são alterados
- imagem do livro: quando o título, a autoria ou a capa são alterados, ou quando uma nova avaliação é feita

### Habilitando imagens de pré-visualização

In order to enable the feature with default settings, you have to uncomment (remove the `#` in front of) the line `ENABLE_PREVIEW_IMAGES=true` in your `.env` file. All the new updating events aforementioned will cause the generation of the corresponding image.

Examples for these images can be viewed on the [feature’s pull request’s description](https://github.com/bookwyrm-social/bookwyrm/pull/1142#pullrequest-651683886-permalink).

### Generating preview images

If you enable this setting after your instance has been started, some images may not have been generated. A command has been added to automate the image generation. In order to prevent a ressource hog by generating **A LOT** of images, you have to pass the argument `--all` (or `-a`) to start the generation of the preview images for all users and books. Without this argument, only the site preview will be generated.

User and book preview images will be generated asynchroneously: the task will be sent to Flower. Some time may be needed before all the books and users have a working preview image. If you have a good book 📖, a kitten 🐱 or a cake 🍰, this is the perfect time to show them some attention 💖.

### Optional settings

So you want to customize your preview images? Here are the options:

- `PREVIEW_BG_COLOR` will set the color for the preview image background. You can supply a color value, like `#b00cc0`, or the following values `use_dominant_color_light` or `use_dominant_color_dark`. These will extract a dominant color from the book cover and use it, in a light or a dark theme respectively.
- `PREVIEW_TEXT_COLOR` will set the color for the text. Depending on the choice for the background color, you should find a value that will have a sufficient contrast for the image to be accessible. A contrast ratio of 1:4.5 is recommended.
- `PREVIEW_IMG_WIDTH` and `PREVIEW_IMG_HEIGHT` will set the dimensions of the image. Currently, the system will work best on images with a landscape (horizontal) orientation.
- `PREVIEW_DEFAULT_COVER_COLOR` will set the color for books without covers.

All the color variables accept values that can be recognized as colors by Pillow’s `ImageColor` module: [Learn more about Pillow color names](https://pillow.readthedocs.io/en/stable/reference/ImageColor.html#color-names).
