#Mapty Workout Application

This project was built following the instructions of Jonas Schedtmann, a Udemy JavaScript course instructor. It simulates a workout application where workouts and their data can be pinned to an interactive map by way of a 3rd party library. 

There are 3 flowcharts provided with this project which provide an abstract look at the overall build and functionality. These can all be found in the files list.

Constructing this project has helped me to learn and practice better the following:
1) Project planning
2) Using Geolocation
3) Working with 3rd party libraries
4) Working with local storage
5) .focus()
6) .closest()
7) guard clauses
8) //prettier-ignore
9) .insertAdjacentHTML()
10) JSON.stringify()

A general walkthrough of the JavaScript code is below.

First access the user location by way of the getCurrentPosition() function which utilizes the built in browser geolocator and takes in two functions, the second being an error function in case the location cannot be determined.
To then display the user location on a map, a 3rd party library is utilized which is known as Leaflet (https://leafletjs.com/).
The latitude and longitude are pulled from the geolocator and then plugged into the leaflet code at .setView() and .marker() which sets the initial view and applies a marker with popup, respectively.
```JavaScript
if (navigator.geolocation)
  navigator.geolocation.getCurrentPosition(
    function (position) {
      const { latitude } = position.coords;
      const { longitude } = position.coords;

      const map = L.map('map').setView([latitude, longitude], 13);

      L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution:
          '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
      }).addTo(map);

      L.marker([latitude, longitude])
        .addTo(map)
        .bindPopup('A pretty CSS3 popup.<br> Easily customizable.')
        .openPopup();
    },
    function () {
      alert('Could not get your position.');
    }
  );
```

To display a marker on the map exactly where a user clicks then an event handler is added to the map. The leaflet library map.on() method is used to add the event listener. Leaflet methods are also called on the .popup to style and appoint the behavior of the marker popups. These methods and more are found in the leaflet documentation.
```JavaScript
      map.on('click', mapEvent => {
        const { lat, lng } = mapEvent.latlng;
        
        L.marker([lat, lng])
          .addTo(map)
          .bindPopup(
            L.popup({
              maxWidth: 250,
              minWidth: 100,
              autoClose: false,
              closeOnClick: false,
              className: 'running-popup',
            })
          )
          .setPopupContent('HUBAAB FUSUS')
          .openPopup();
```


Whenever a user clicks on the map, the workout form should be displayed with the new workout added. Once the submit button is clicked, the map will then set a marker to that location and clear the input fields. map and mapEvent are set as global variables so that they are made available throughout the scope chain.
```JavaScript
      let map, mapEvent;

      map.on('click', mapE => {
        mapEvent = mapE;
        form.classList.remove('hidden');
        inputDistance.focus();
      });
      
      form.addEventListener('submit', e => {
  e.preventDefault();
  const { lat, lng } = mapEvent.latlng;
  L.marker([lat, lng])
    .addTo(map)
    .bindPopup(
      L.popup({
        maxWidth: 250,
        minWidth: 100,
        autoClose: false,
        closeOnClick: false,
        className: 'running-popup',
      })
    )
    .setPopupContent('Gafilkafish!')
    .openPopup();
    
    //Clear input fields
    inputDistance.value =
    inputDuration.value =
    inputCadence.value =
    inputElevation.value =
      '';
});
```


To toggle between the Cadence and Elevation options an event listener is added to the inputType to listen for the change event. This allows the hidden class to be toggled using the .closest() method which seeks out the parent element with the same name - form__row in this case. By toggling both one will always be visible.
```JavaScript
inputType.addEventListener('change', e => {
  inputElevation.closest('.form__row').classList.toggle('form__row--hidden');
  inputCadence.closest('.form__row').classList.toggle('form__row--hidden');
});
```


Check the script.js file to see how the previous functions were refactored into a class. Next a parent Workout class was created.
```JavaScript
class Workout {
  date = new Date();
  id = (new Date() + '').slice(-10);

  constructor(coords, distance, duration) {
    this.coords = coords;
    this.distance = distance;
    this.duration = duration;
  }
}
```


The child classes were created next, one for running and one for cycling. super() sets the this keyword.
```JavaScript
class Running extends Workout {
  constructor(coords, distance, duration, cadence) {
    super(coords, distance, duration);
    this.cadence = cadence;
    this.calcPace();
  }
  calcPace() {
    //min/km
    this.pace = this.duration / this.distance;
    return this.pace;
  }
}

class Cycling extends Workout {
  constructor(coords, distance, duration, elevationGain) {
    super(coords, distance, duration);
    this.elevationGain = elevationGain;
  }
  calcSpeed() {
    //   km/hr
    this.calcSpeed();
    this.speed = this.distance / (this.duration / 60);
    return this.speed;
  }
}
```


The new classes created are then used to create new workouts from data inputted to the UI form. Data inputted needs to be validated and then a new object created for the new workout. Lastly, render the new workout to the UI, list and map.
To do this the newWorkout() function inside of the App class is added on to as well as the renderWorkout() function is created.
```JavaScript
  _newWorkout(e) {
    const validInputs = (...inputs) =>
      inputs.every(inp => Number.isFinite(inp));

    const allPositive = (...inputs) => inputs.every(input => input >= 0);

    e.preventDefault();
    //Get data from form
    const type = inputType.value;
    const distance = Number(inputDistance.value);
    const duration = +inputDuration.value;
    const { lat, lng } = this.#mapEvent.latlng;
    let workout;

    //If workout running, create running object
    if (type === 'running') {
      const cadence = +inputCadence.value;
      //Check that data is valid
      if (
        // !Number.isFinite(distance) ||
        // !Number.isFinite(duration) ||
        // !Number.isFinite(cadence)
        !validInputs(distance, duration, cadence) ||
        !allPositive(distance, duration, cadence)
      )
        return alert('Inputs have to be positive numbers!');

      workout = new Running([lat, lng], distance, duration, cadence);
    }

    //If workout cycling, create cycling object
    if (type === 'cycling') {
      //Check that data is valid
      const elevation = +inputElevation.value;
      if (
        !validInputs(distance, duration, elevation) ||
        !allPositive(distance, duration)
      )
        return alert('Inputs have to be positive numbers!');

      workout = new Cycling([lat, lng], distance, duration, elevation);
    }

    //Add new object to workout array
    this.#workouts.push(workout);

    //Render workout on map as marker
    this._renderWorkoutMarker(workout);
    //Render workout on list
    this._renderWorkout(workout);
    //Hide form + clear input fields
    inputDistance.value =
      inputDuration.value =
      inputCadence.value =
      inputElevation.value =
        '';
  }
  
    _renderWorkout(workout) {
    const html = `
    <li class="workout workout--${workout.type}" data-id="${workout.id}">
      <h2 class="workout__title">${workout.description}</h2>
      <div class="workout__details">
        <span class="workout__icon">${
          workout.type === 'running' ? '🏃‍♂️' : '🚴‍♂️'
        }</span>
        <span class="workout__value">${workout.distance}</span>
        <span class="workout__unit">km</span>
      </div>
      <div class="workout__details">
        <span class="workout__icon">⏱</span>
        <span class="workout__value">${workout.duration}</span>
        <span class="workout__unit">min</span>
      </div>`;
  }
  
```


The description displayed by the popup markers is set by way of the setDescription() function. The first letter of the type ('running' 'cycling') is capitalized then the remaining letters after sliced and attached to the uppercase first letter. [this.date.getMonth()] returns a 0-11 array which is used to render the month. Same for day.
```JavaScript
  _setDescription() {
    const months = [
      'January',
      'February',
      'March',
      'April',
      'May',
      'June',
      'July',
      'August',
      'September',
      'October',
      'November',
      'December',
    ];

    this.description = `${this.type[0].toUppercase()}${this.type.slice(1)} on ${
      months[this.date.getMonth()]
    }${this.date.getDate()}`;
  }
```


The HTML is dyanmically edited using template literals and then inserted to the DOM using the insertAdjacentHTML method. Then the form is hidden.
```JavaScript
...
     <div class="workout__details">
        <span class="workout__icon">⛰</span>
        <span class="workout__value">${workout.cadence}</span>
        <span class="workout__unit">m</span>
      </div>
    </li>`;

    form.insertAdjacentHTML('afterend', html);
    
      _hideForm() {
    inputDistance.value =
      inputDuration.value =
      inputCadence.value =
      inputElevation.value =
        '';
    form.style.display = 'none';
    form.classList.add('hidden');
    setTimeout(() => (form.style.display = 'grid'), 1000);
  }
```


To format the display of the marker popups the setPopupContent() function is edited.
```JavaScript
      .setPopupContent(
        String(
          `${workout.type === 'running' ? '🏃‍♂️' : '🚴‍♂️'} ${workout.description}`
        )
```


In order to move to a marker on a list item click event the moveToPopup() function was created.
```JavaScript
  _moveToPopup(e) {
    const workoutEl = e.target.closest('.workout');
    if (!workoutEl) return;
    const workout = this.#workouts.find(
      work => work.id === workoutEl.dataset.id
    );
    console.log(workout);
    this.#map.setView(workout.coords, 13, {
      animate: true,
      pan: {
        duration: 1,
      },
    });
  }
```


To make the workout data persist across page reloads the local storage API is utilized whenever a new workout is submitted by way of the setLocalStorage() function.
```JavaScript
  _setLocalStorage() {
    localStorage.setItem('workouts', JSON.stringify(this.#workouts));
  }
```



To reload old workouts stored inside of local storage and then render them to the map and UI list the getLocalStorage() function was constructed.
```JavaScript
  _getLocalStorage() {
    const data = JSON.parse(localStorage.getItem('workouts'));

    if (!data) return;
    //sets workout array to workouts in LS(if any)
    this.#workouts = data;
    //renders workouts
    this.#workouts.forEach(work => {
      this._renderWorkout(work);
    });
  }
```

To remove items from local storage the reset() function was constructed.
```JavaScript
  reset() {
    localStorage.removeItem('workouts');
    location.reload();
  }
```

***Walkthrough finished
