PackageFactory:BaseJump
=======================

Experimental package to explore the feasibility of basic symfony style 
routing and controller actions in Neos/FLow. 

If you sometimes dislike property mapping, validation and automatic views 
this may be valuable for you! Otherwise this is probably dangerous stuff.

BE SURE WHETHER THIS IS WHAT YOU WANT AND THAT YOU UNDERSTAND THE CONSEQUENCES

The package provides the BaseJumpController base class as an alternative 
to the Flow ActionController. The BaseJumpController will pass the 
incoming ActionRequest and ActionResponse directly to the action method.
The method can use the ActionRequest to access the requested arguments and 
modify the ActionResult. Alternatively it may also return a PSR7 Response.

ATTENTION: This approach totally circumvents all the convenience and security 
feature flow usually offers via property mapping and validation. Also no 
view is instantiated unless you do so in the controller.


ExampleController.php
```
<?php
namespace Vendor\Package\Controller;

use PackageFactory\BaseJump\Controller\BaseJumpController;
use Neos\Flow\Mvc\ActionRequest;
use Neos\Flow\Mvc\ActionResponse;
use Psr\Http\Message\ResponseInterface;
use GuzzleHttp\Psr7\Response;

class ExampleController extends BaseJumpController
{
    /**
     * @param ActionRequest $request
     * @param ActionResponse $response
     * @return ResponseInterface|null
     */
    public function exampleAction (ActionRequest $request, ActionResponse $response): ?ResponseInterface
    {
        return new Response(
            200,
            ['x-foo' => 'bar'],
            'action result'
        );
    }
}
```

To actually access the controller action a route has to be defined to map
url pathes and http methods controller action methods. 

https://flowframework.readthedocs.io/en/stable/TheDefinitiveGuide/PartIII/Routing.html

Routes.yaml:
```
-
  name: 'Test Foo'
  uriPattern: 'exampple/{name}'
  httpMethods: ['GET', 'HEAD']
  defaults:
    '@package':    'Vendor.Package'
    '@controller': 'Example'
    '@action':     'example'
```

The last required part is a policy that allows access the action
from the outside. Otherwise the flow security will intercept the 
request.

https://flowframework.readthedocs.io/en/stable/TheDefinitiveGuide/PartIII/Security.html#authorization

Policy.yaml
```
privilegeTargets:

  'Neos\Flow\Security\Authorization\Privilege\Method\MethodPrivilege':

    'Vendor.Package:ExampleController':
      matcher: 'method(Vendor\Package\Controller\ExampleController->(example)Action())'

roles:

  'Neos.Flow:Everybody':
    privileges:
      -
        privilegeTarget: 'Vendor.Package:ExampleController'
        permission: GRANT
```
