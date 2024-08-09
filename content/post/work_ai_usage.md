+++
title = 'Using AI for Work'
date = 2024-08-09T09:01:56-05:00
draft = false
tags = ["ai", "go", "gemini", "google", "testing", "tdd", "vscode"]
summary = "A simple example of how AI was helpful for a small work problem."
+++
# Using AI for Work

So where I currently work, Google is the *thing*, so our AI/LLM of choice is [Gemini](https://gemini.google.com). I recently was mentoring an intern who was working through a problem with a Go program. The short version of the Go program is that it does a bunch of http "GET" requests to check the status of a bunch of services. So the question was how to correctly write tests for this program such that the code doesn't actually do *real* http requests.

This is where I think AI is of real use. Using the [Gemini Code Assist extension in VSCode](https://cloud.google.com/code/docs/vscode/write-code-gemini), it was possible to then just ask it to help write tests for the code in question. It suggested several things, but most importantly it showed how to do dependency injection for the function(s) in question and showed how to create an interface appropriate for our use of the http library.

{{< callout note >}}
Like a lot of VSCode AI integrations, this requires an account of sorts. In our case, the Gemini extension is linked to our GCP account and designated *project* where Gemini (formerly *Duet*) is enabled as something we as an organization can use. This is also for security purposes where Google is able to assure a company that their information will remain private.
{{< /callout >}}

Then for actually testing, it kindly suggested a structure and functions for use in the tests.

```go
// MockHttpClient implements the HttpClient interface for testing.
type MockHttpClient struct {
	Responses map[string]*http.Response
}

// Get returns a pre-defined response based on the URL.
func (m *MockHttpClient) Get(url string) (*http.Response, error) {
	if response, ok := m.Responses[url]; ok {
		return response, nil
	}
	return nil, fmt.Errorf("no mock response found for URL: %s", url)
}

// Helper function to create mock responses
func createMockResponse(version string) *http.Response {
	mockResponse := httptest.NewRecorder()
	mockResponse.Write([]byte(fmt.Sprintf(`{"version": "%s"}`, version)))
	mockResponse.Code = http.StatusOK
	return mockResponse.Result()
}
```

Where it fell a bit short was some overly repetitive code in actual test functions. I had to request it create a constructor function that supplied some commonly used URLs for the tests so that we could create a fresh mock client easily in each test without a bunch of lines getting repeated:

```go
func NewMockHttpClient() *MockHttpClient {
	return &MockHttpClient{
	 	Responses: map[string]*http.Response{
	  		"https://one-dev.example.com/version":  createMockResponse("0.26.0"),
	  		"https://one-qa.example.com/version": createMockResponse("0.26.0"),
	  		"https://two-dev.example.com/version":    createMockResponse("4.3.0"),
	  		"https://two-qa.example.com/version":   createMockResponse("4.2.2"),
	  		"https://two-prod.example.com/version":   createMockResponse("4.2.1"),
		},
	}
}
```

Obviously, if some additional URL was needed, you could create whatever you needed but for 95% of all the tests, this standard set of URLs was fine. All that was necessary was to call the function being tested with this mock dependency in place of the normal `http.Client` that would be "injected".

A couple of things of note: Gemini did not rewrite every test or actually change the code in place. But what it did do was explain how dependency injection was going to be used, showed the relevant bits and showed how a couple of the existing tests could be changed to take advantage of the concept. To me, this sort of pointing-in-the-right-direction use of AI is fantastic. Forget how to do something or unsure which way to go next? This seems more effective than a Google search.

The other thing Gemini does when integrated with VSCode is to provide inline suggestions as you type. After changing code in some of the tests to create a new `mockClient` via `mockClient := NewMockHttpClient()`, the Gemini suggestion would start showing that line as a completion immediately when you're about to type in a similar place in a different test. The inline completions are a little jarring at first, and there is an option to turn them off, but once you get used to the fact that it might provide something really useful, it's pretty neat.

Again, though, discernment is necessary as a developer. In that previous example, a developer should realize, that unless some change is necessary to the `mockClient`, you don't really need to even create a `mockClient` separately, just call the function being tested with `NewMockHttpClient()` as a parameter instead. Gemini doesn't realize that that can be done even when I asked it to help eliminate unnecessary lines of code.

Overall, though, this really does feel like a super advanced form of code assist. It doesn't eliminate you having to know what you're doing, but as a tool, it can really help speed up your work and get you unstuck in situations where you're having trouble figuring out what to do next.

{{< callout important "obvious callout" >}}
If you're a hardcore TDD person, this example is likely triggering. Writing tests after code! But I think this is a realistic situation for utility scripts/programs like this. Someone new to a language is just trying to make something work and seeing how it works. They figure out how the pieces (libraries) fit together and work and then, "oh, how do we have tests for this?" After this whole interchange, future uses of the `net/http` library in Go, a developer will be *more* likely to either do TDD or at least code with tests and dependency injection in mind.
{{< /callout >}}