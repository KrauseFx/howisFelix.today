# [howisFelix.today?](https://howisFelix.today)

<img src="https://raw.githubusercontent.com/KrauseFx/howisfelix.today/master/screenshots/Desktop-1.png" />

## See the result on <a href="https://howisFelix.today/">howisFelix.today</a>

<img src="https://raw.githubusercontent.com/KrauseFx/howisFelix.today/master/screenshots/Desktop-1.png" />

<h2 align=center>See the result on <a href="https://howisFelix.today/">howisFelix.today</a></h2>
<p />


<table>
  <tr>
    <td>
      <img src="https://raw.githubusercontent.com/KrauseFx/howisFelix.today/master/screenshots/iPhone-1.png" />
    </td>
    <td>
      <img src="https://raw.githubusercontent.com/KrauseFx/howisFelix.today/master/screenshots/iPhone-2.png" />
    </td>
    <td>
      <img src="https://raw.githubusercontent.com/KrauseFx/howisFelix.today/master/screenshots/iPhone-3.png" />
    </td>
  </tr>
</table>

---

## FxLifeSheet

The data is collected and stored by the [KrauseFx/FxLifeSheet](https://github.com/KrauseFx/FxLifeSheet) project.

## Development

```
bundle install
```

```
bundle exec jekyll s
```

Whenever you update `data.erb` you need to run

```
bundle exec ruby graphs/update.rb
```

to speed things up and skip converting images on the fly, you can use

```
SKIP=1 be ruby graphs/update.rb
```

## Backend

The server code is on https://github.com/KrauseFx/whereisfelix.today-backend, which used to also include an Angular-based frontend, but now got replaced by a simple Jekyll application
