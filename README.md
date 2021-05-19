[![npm](https://img.shields.io/npm/v/grid-angular-adapter.svg)]( https://www.npmjs.com/package/grid-angular-adapter) [![CircleCI](https://circleci.com/gh/gridgrid/grid-angular-adapter.svg?style=shield)](https://circleci.com/gh/gridgrid/grid-angular-adapter)  [![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg)](https://github.com/semantic-release/semantic-release) [![Commitizen friendly](https://img.shields.io/badge/commitizen-friendly-brightgreen.svg)](http://commitizen.github.io/cz-cli/)

Grid Angular Adapter
===

Tools for using Grid with angular (1.0 currently) NOTE: this project is no longer maintained. If you want an angular adapter surely it will look very different from angular 1 and you would be welcome to take over this repo :)

Installation
===
`npm install --save grid-angular-adapter`


the three srvcs currently exposed can be injected

`function(GridSrvc, GridBuilderSrvc, GridDecoratorSrvc){}`


if not using angular 

`var core = require('grid/src/modules/core')`


**angular directive decorator**
``` javascript
angular.module('myCoolGrid', [require('grid')])

  .controller('MyGridCtrl', function($scope, GridSrvc, GridDecoratorSrvc) {
      var grid = GridSrvc.core();

      // do row col and datamodel setup...

      var top = 20,
          left = 10,
          height = 2,
          width = 2,
          unit = 'cell',
          space = 'virtual';
      var d = grid.decorators.create(top, left, height, width, unit, space);

      // return some element
      d.render = function() {
          var scope = $scope.$new();
          scope.interestingData = 'INTERESTING DATA!!!';
          return GridDecoratorSrvc.render({
              $scope: scope,
              template: '<my-directive data="interestingData"></my-directive>',
              events: true
          });
      };
      grid.decorators.add(d);
  });
```

This will put your compiled directive in a box that spans from `row 20-22` and `col 10-12`, and moves appropriately with the scroll. The `events` flag lets the grid know that this decorator is interactable and should receive mouse events. (Otherwise pointer events are set to none.) The GridDecoratorSrvc takes care of destroying the scope and properly removing elements to avoid memory leaks with angular. You definitely should use it for any angular decorators.


## Builders
Builders help you to get html into the actual cells of a given row or column instead of the text that would have been rendered.


**basic builders**
``` javascript
var builder = grid.colModel.createBuilder(function render() {
    return angular.element('<a href="#"></a>')[0];
}, function update(elem, ctx) {
    grid.viewLayer.setTextContent(elem, ctx.data.formatted);
    return elem;
});
var colDescriptor = grid.colModel.create(builder);
```

have `<a>` tags in your cells for all the rows in that column

**angular cell builder**

``` javascript
angular.module('myCoolGrid', [require('grid')])

.controller('MyGridCtrl', function($scope, GridSrvc, GridBuilderSrvc) {
    var grid = GridSrvc.core();

    // do row col and datamodel setup...

    grid.colModel.create(grid.colModel.createBuilder(function render(ctx) {
        return GridBuilderSrvc.render($scope, '<my-directive data="interestingData"></my-directive>');
    }, function update(elem, ctx) {
        var scope = angular.element(elem).scope();
        scope.interestingData = ctx.data.formatted;
        scope.$digest();
        return elem;
    }));
});
```

The GridBuilderSrvc handles destroying the scope and properly removing the elements to prevent memory leaks.

Note: it's important for the update function of a builder to be extremely fast. Call `scope.$digest` not `scope.$apply`, and  use `grid.viewLayer.setTextContent` not `elem.innerHTML` where possible

