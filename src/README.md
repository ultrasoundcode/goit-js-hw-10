```javascript
// Импортированы стили, функция задержки и библиотека для уведомлений
import './css/styles.css';
import debounce from 'lodash.debounce';
import Notiflix from 'notiflix';
// Импортирована функция загрузки данных из другого модуля
import { fetchCountries } from './fetchCountries';

// Задержка ввода
const DEBOUNCE_DELAY = 300;
// Поисковое поле и переменные общей информации о стране и списка стран
const inputEl = document.querySelector('#search-box');
const countryInfoEl = document.querySelector('.country-info');
const countryListEl = document.querySelector('.country-list');

// Функция обработчика поиска
function getCountries() {
  // Получает данные из поля ввода и очищает информацию о странах
  const inputData = inputEl.value.trim();
  countryInfoEl.innerHTML = '';
  countryListEl.innerHTML = '';

  // Если поле не пустое, запускается функция получения данных с сервера
  if (inputData !== '') {
    fetchCountries(inputData)
      .then(data => {
        inputEl.classList.remove('error');
        // Если запрос вернул 0 результатов, то ничего не делается
        if (data.length === 0) {
          return;
          // Если вернулся всего один результат, отображается подробная информация о стране
        } else if (data.length === 1) {
          renderCountry(data);
          // Если вернулось от двух до десяти результатов, список всех стран выводится на экран
        } else if (data.length >= 2 && data.length <= 10) {
          renderCountries(data);
          // Если вернулось более 10 результатов, сообщение о том, что необходимо ввести более точное название страны
        } else if (data.length > 10) {
          Notiflix.Notify.info(
            'Too many matches found. Please enter a more specific name.'
          );
        }
      })
      // Если произошла ошибка, выводит уведомление и добавляет класс-индикатор ошибки к полю
      .catch(() => {
        Notiflix.Notify.failure('Oops, there is no country with that name');
        inputEl.classList.add('error');
      });
  }
}

// Функция рендера списка стран
function renderCountries(countries) {
  const markupCountries = countries
    .map(country => {
      return `<li><img src=${country.flags.svg} alt="flag, ${country.name.official}" width=30, height=auto><p>${country.name.official}</p></li>`;
    })
    .join('');
  countryListEl.innerHTML = markupCountries;
}

// Функция рендера более подробной информации о выбранной стране
function renderCountry(countries) {
  const markupCountry = countries
    .map(country => {
      return `<div class="country-card"><img src=${
        country.flags.svg
      } alt="flag, ${country.name.official}" width=40, height=auto><h2>${
        country.name.official
      }</h2><p><span class="title">Capital:</span> ${
        country.capital
      }</p><p><span class="title">Population:</span> ${
        country.population
      }</p><p><span class="title">Languages:</span> ${Object.values(
        country.languages
      )}</p></div>`;
    })
    .join('');
  countryInfoEl.innerHTML = markupCountry;
}

// Обработчик ввода с задержкой при помощи функции debounce(lodash)
inputEl.addEventListener(
  'input',
  debounce(event => getCountries(event), DEBOUNCE_DELAY)
);

// Обработчик нажатия на элемент списка стран для дальнейшего поиска
countryListEl.addEventListener('click', chooseCard);

// Функция выбора страны после нажатия на ее название из списка стран
function chooseCard(event) {
  inputEl.value = event.target.innerText.trim();
  const card = inputEl.value;
  getCountries(card);
  inputEl.value = '';
}
```
