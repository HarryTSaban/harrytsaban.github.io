## Software Engineer working in Cloud, Apps, Algos and Infra

# Chasing the Milliseconds

When working with a system of microservices, distributed across a ton of AWS Lambdas and Step Functions it's important to take a pause every now and then to consider the performance of your system as a whole. Whilst very cost-effective for scaling, Lambdas can be a little limiting when you're trying to keep your response times snappy (especially in a python-heavy stack). Fortunately there are a few little tips and tricks you can use that take advantage of the Lambda ecosystem:

1. **Always cache your clients:** We managed to shave off around 200ms/call on one of our endpoints when we stopped instantiating a new boto client for every single S3 call we had to make. You can see similar improvements by insuring that you don't repeatedly instantiate database or HTTP clients.
2. **X-ray is your best friend:** Once you set up [AWS X-ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html) to track all of your inter-service/DB/HTTP calls you can get a ton of info on where the pain-points are in your services.
3. **Make the most of the INIT phase:** A [quirk of Lambdas](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtime-environment.html) is that you are given 10s of set up time with some extra CPU which you can take advantage of if you perform some actions before starting your handler function. Use this for getting your clients and sessions running, or preloading files into memory so that you don't have to do it during call-time.
    1. **Explore SnapStart:** Heavy python lambdas can have some heinously slow cold-start times, [Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html) is supported from Python 3.12 onwards and reduced one of our services cold-start times from 5s to 500ms! It's not free, but provides a nice avenue for keeping critical services responsive.
4. **Keep it simple!** As we build up a system of microservices that interact with one another, it's very easy for complexity and back-and-forth calls to start building up. Remember that every call out to another lambda adds overhead, so do what you can to keep things contained.

The upside of chasing down performance improvements is not just a vastly improved user experience, but also (with the exception of SnapStart) lower running costs! Remember that AWS Lambda charges by the GB ms, so shave off those milliseconds!
