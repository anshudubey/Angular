# Angular
new line

 Angular 17 automatically removes component-associated styles from the DOM when the component is destroyed. In Angular 15, these styles persisted, which some applications might have relied on implicitly.

**************************************************************************
Install dependencies in each app (npm install or yarn).
Start apps in separate terminals:
# Angular Microfrontend
cd angular-app
ng serve --port 4201

# React Microfrontend
cd react-app
npx webpack serve

# Container Host App
cd container-app
ng serve --port 4200

