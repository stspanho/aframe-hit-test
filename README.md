# WebXR hit-testing example

In this example I've used the Spinosaurus example by Klaus Weidner (https://glitch.com/edit/undefined#!/xr-spinosaurus)
and implemented a hit-testing component for A-Frame.

The component uses the new WebXR hit-testing feature. You can test this with an Android phone and a Chrome (Beta) v81+ browser.

You can read more about this feature here: https://immersive-web.github.io/hit-test/

![](example.gif)

## The component

```
AFRAME.registerComponent('ar-hit-test', {
			init: function () {
				this.xrHitTestSource = null;
				this.viewerSpace = null;
				this.refSpace = null;

				this.el.sceneEl.renderer.xr.addEventListener('sessionend', (ev) => {
					this.viewerSpace = null;
					this.refSpace = null;
					this.xrHitTestSource = null;
				});
				this.el.sceneEl.renderer.xr.addEventListener('sessionstart', (ev) => {
					let session = this.el.sceneEl.renderer.xr.getSession();

					let element = this.el;
					session.addEventListener('select', function () {
						let position = element.getAttribute('position');

						document.getElementById('dino').setAttribute('position', position);
						document.getElementById('light').setAttribute('position', {
							x: (position.x - 2),
							y: (position.y + 4),
							z: (position.z + 2)
						});
					});

					session.requestReferenceSpace('viewer').then((space) => {
						this.viewerSpace = space;
						session.requestHitTestSource({space: this.viewerSpace})
								.then((hitTestSource) => {
									this.xrHitTestSource = hitTestSource;
								});
					});

					session.requestReferenceSpace('local-floor').then((space) => {
						this.refSpace = space;
					});
				});
			},
			tick: function () {
				if (this.el.sceneEl.is('ar-mode')) {
					if (!this.viewerSpace) return;

					let frame = this.el.sceneEl.frame;
					let xrViewerPose = frame.getViewerPose(this.refSpace);

					if (this.xrHitTestSource && xrViewerPose) {
						let hitTestResults = frame.getHitTestResults(this.xrHitTestSource);
						if (hitTestResults.length > 0) {
							let pose = hitTestResults[0].getPose(this.refSpace);

							let inputMat = new THREE.Matrix4();
							inputMat.fromArray(pose.transform.matrix);

							let position = new THREE.Vector3();
							position.setFromMatrixPosition(inputMat);
							this.el.setAttribute('position', position);
						}
					}
				}
			}
		});
```