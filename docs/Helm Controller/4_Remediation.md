# Remediation

You need to define a remediation strategy for your releases as the default behavior of flux is `` to do nothing `` -_- .

There are two strategies :
- Failed install : Perform an uninstall action of the failed Helm Release
- Failed upgrade : Perform a rollback, unless the strategy is set to uninstall


Check more about strategies here :
- https://app.pluralsight.com/course-player?clipId=5761b0b9-abf3-43ea-bebb-279598bb7914

And to check a demo, you can find it here :
- https://app.pluralsight.com/course-player?clipId=efcd61e7-f765-4820-8161-2b3887eefc22


## References
- https://app.pluralsight.com/course-player?clipId=5761b0b9-abf3-43ea-bebb-279598bb7914
- https://app.pluralsight.com/course-player?clipId=efcd61e7-f765-4820-8161-2b3887eefc22