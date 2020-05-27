---
layout: post
title:  "Spring Security를 사용한 로그인 인증"
date:   2020-05-27
image: 'img024_01.PNG'
categories: [Integration]
tags: [spring,boot,postgresql,setting,develoment,environment,authentication,인증,security,보안,스프링,부트,뷰,세팅,개발,환경,개발환경]
---


안녕하세요. 오랫만에 글을 쓰게 되었습니다.

지난 글인 Spring Boot + Vue.js + PostgreSQL 개발 환경 세팅의 일환으로 Spring Security를 사용하여 로그인하는 부분을 써보겠습니다.

* [Spring Boot + Vue.js + PostgreSQL 개발환경 세팅(Overview)](/blog/SB-VJ-PS-Setting/)

이번 글 작성을 위해서 가장 참고가 많이 된 글은 다음과 같습니다.  다시한번 감사의 말을 전하겠습니다.

* [BezKoder - Spring Boot Token based Authentication with Spring Security & JWT](https://bezkoder.com/spring-boot-jwt-authentication/)



#### 1. 서론

먼저 잡소리부터 좀 하겠습니다.

이번에 회사 프로젝트로 Spring Boot + Vue.js를 사용해서 웹 애플리케이션을 신규 구축해야 하는 과제를 맡게 되었습니다.
이에 따라 프로젝트 환경 설정도 제가 진행을 하게 되었는데요.
사실 신규 구축이라기 보다는 기존에 구축되어 있는 시스템을 참고해서 Migration을 하는 것이기도 했습니다.

원래는 Spring Security를 사용해서 체계적인 회원관리 시스템 및 인증까지 구현을 하려고 계획했습니다만, 안타깝게도 기존 시스템에서의 회원관리 시스템을 그대로 가지고 오라는 지시가 있어서
울며 겨자먹기로 진행을 하게 되었습니다.

결론부터 말하자면, 제가 이번 글에 소개해드릴 내용대로 원래는 구현하려고 했었으나 결국은 PostgreSQL의 Function을 사용해서 인증을 구현하기로 나타난 것이죠.
해당 구현은 Spring Security를 사용하지도 않고, DB Function 자체 구현이기 때문에 보안 측면에서 더욱 강력하다고 보기도 어렵습니다만 어쨌든 그리 되었습니다.

그래서 제가 구현해 놓은 환경을 이대로 버리기도 아깝고 해서 이 글에서나마 소개하고자 합니다.

여러분들은 개발을 진행할 때 자기 소신대로 밀고 붙이시는 것을 되도록 권장합니다.
저는 그리 하지 못해서 매우 개인적으로 안타깝고 때로는 화가 나기도 했지만 어찌되었든 돈 벌어먹고 살려면 하라는 대로 했을 뿐.
이런 사례가 가급적 자주 일어나지는 않았으면 합니다.


### 2. 개요

일단 제일 중요한 프로세스는 아래와 같습니다. 아래 프로세스는 제가 그린게 아니라 BezKoder 님의 글에 있는 이미지이니 참고 바랍니다.

![Process]({{ 'https://bezkoder.com/wp-content/uploads/2019/10/spring-boot-authentication-jwt-spring-security-flow.png'}})

이것으로 설명이 다 되겠네요. 자세한 설명은 아래에서 하나씩 다루겠습니다.
일반적인 순서면 먼저 무엇을 만들어야 할 것인지를 언급하고 하나하나씩 보는 순서여야 하지만, 여기서는 이례적으로 Controller 코드를 보여주고 파생되는 Object를 생성하는 형태로 진행할게요.
로그인/회원가입 프로세스의 흐름대로 따라가기 위함이니 참고 바랍니다.

모든 코드는 BezKoder 님의 글의 코드를 참고하겠습니다.


### 3. Spring Security 수행 동작 및 환경 설정

먼저 Spring Security 구조를 보겠습니다. 사진은 역시 BezKoder님 글의 이미지입니다.

![Security Process]({{ 'https://bezkoder.com/wp-content/uploads/2019/10/spring-boot-authentication-spring-security-architecture.png'}})

Spring Security(이하 Security)를 pom.xml에서 추가하게 되면 웹페이지에 접근할 때 Security를 제일 먼저 거치는 동작을 수행합니다.
그래서 Security만 설치하고 아무것도 없는 웹 애플리케이션을 실행하면 로그인 페이지가 나타날 것이고요.

![Security Login]({{ '/assets/img/img024_01.PNG' | prepend: site.baseurl }})

물론 우리는 저 페이지를 그대로 사용하지는 않습니다. 그렇기 때문에 Security에서 환경 설정을 해야 되겠죠.

Security에서 환경설정을 하는 부분은 WebSecurityConfigurerAdapter 클래스를 사용하며, 실제 구현은 이를 상속한 클래스로 이루어집니다.
해당 클래스는 다음과 같습니다.

{% highlight Java %}
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(
		// securedEnabled = true,
		// jsr250Enabled = true,
		prePostEnabled = true)
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
	@Autowired
	UserDetailsServiceImpl userDetailsService;

	@Autowired
	private AuthEntryPointJwt unauthorizedHandler;

	@Bean
	public AuthTokenFilter authenticationJwtTokenFilter() {
		return new AuthTokenFilter();
	}

	@Override
	public void configure(AuthenticationManagerBuilder authenticationManagerBuilder) throws Exception {
		authenticationManagerBuilder.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
	}

	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}

	@Bean
	public PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http.cors().and().csrf().disable()
			.exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
			.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
			.authorizeRequests().antMatchers("/api/auth/**").permitAll()
			.antMatchers("/api/test/**").permitAll()
			.anyRequest().authenticated();

		http.addFilterBefore(authenticationJwtTokenFilter(), UsernamePasswordAuthenticationFilter.class);
	}
}
{% endhighlight %}

여기서 제일 처음에 보는 부분은 맨 아래의 configure(HttpSecurity) 메소드입니다.
접근하는 페이지가 어떤 페이지인가.

위 코드에 의하면 /api/auth/ 경로의 모든 파일은 접근을 허용합니다.
하지만 /api/auth/ 외 다른 경로는 그냥 허용하게 두지는 않는다는 뜻입니다.

만약에 접근을 원한다면 인증 절차를 거치게 되겠죠.

그 외 다른 메소드의 경우는 Bean을 생성해주는 역할로, AuthTokenFilter, AuthenticationManager, PasswordEncoder를 생성하는 부분입니다.
간단히 짚어보면 다음과 같습니다.

* AuthTokenFilter
  + OncePerRequestFilter로부터 상속받은 객체로, 직접 사용자가 선언해야 합니다. 
  + 이 부분에서는 로그인을 시도할 때 객체를 생성하며, Security의 내장 객체(UserDetails)를 가지고 인증을 수행합니다.
  
* AuthenticationManager
  + Security 내장객체인 AuthenticationManager를 반환합니다. 별도로 선언은 하지 않아도 되며, 인증 수행을 위한 동작을 나타냅니다.
  
* PasswordEncoder
  + 역시 내장객체로, 비밀번호 암호화를 담당합니다.
  
환경 설정과 관련된 부분은 여기까지입니다.


### 4. Request Filter 정의

#### 4-1) AuthTokenFilter 정의

이제 다시 위 Security의 프로세스를 보겠습니다.
프로세스를 보면 Request가 들어올 경우 OncePerRequestFilter로 오는 것으로 화살표가 표시되어 있습니다.

Security에서는 모든 웹페이지에 대한 요청이 들어올 때 OncePerRequestFilter로 오는 것으로 설계가 되어 있기 때문입니다.

OncePerRequestFilter에서 수행하는 동작은 웹페이지에 접근할 때 현재 로그인이 되어 있는 상태인지에 대한 부분을 검사합니다.
물론 이에 대한 검사를 Controller에서 수행할 수도 있지만, Security에서는 Request를 한 곳에서 처리하기 위한 기능을 가지고 있다고 보면 됩니다.

하지만 로그인 인증 여부를 확인하기 위해서는 실제 로그인 정보에 대한 접근이 필요하기 때문에 이를 구현하기 위한 클래스가 필요합니다.
여기에서는 그 클래스를 AuthTokenFilter로 하였으며, 아래와 같이 나타냅니다.

{% highlight Java %}
public class AuthTokenFilter extends OncePerRequestFilter {
	@Autowired
	private JwtUtils jwtUtils;

	@Autowired
	private UserDetailsServiceImpl userDetailsService;

	private static final Logger logger = LoggerFactory.getLogger(AuthTokenFilter.class);

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		try {
			String jwt = parseJwt(request);
			if (jwt != null && jwtUtils.validateJwtToken(jwt)) {
				String username = jwtUtils.getUserNameFromJwtToken(jwt);

				UserDetails userDetails = userDetailsService.loadUserByUsername(username);
				UsernamePasswordAuthenticationToken authentication = new UsernamePasswordAuthenticationToken(
					userDetails, null, userDetails.getAuthorities());
				authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

				SecurityContextHolder.getContext().setAuthentication(authentication);
			}
		} catch (Exception e) {
			logger.error("Cannot set user authentication: {}", e);
		}

		filterChain.doFilter(request, response);
	}

	private String parseJwt(HttpServletRequest request) {
		String headerAuth = request.getHeader("Authorization");

		if (StringUtils.hasText(headerAuth) && headerAuth.startsWith("Bearer ")) {
			return headerAuth.substring(7, headerAuth.length());
		}

		return null;
	}
}
{% endhighlight %}

객체 구조는 조금 복잡하긴 하지만 인증 시 수행하기 위한 doFilterInternal() 메소드를 오버라이드를 선언하는 것이 주요 내용입니다. 구조는 다음과 같습니다.

* 1) parseJwt() 메소드 호출: 로그인 정보에 대한 Token을 "Bearer"라는 접두사를 달았고, 접두사를 제외한 나머지 정보를 가져옵니다.
* 2) jwtUtils.validateJwtToken()  메소드 호출 및 비교: jwtUtils 클래스에서 Token 검사를 합니다. jwtUtils는 사용자가 선언해야 하며, 아래에서 다루겠습니다.
* 3) UserDetailsServiceImpl 객체 생성
  + 로그인 정보 객체로, 사용자 이름을 가지고 실제 등록된 사용자인지를 확인한 후 로그인 정보를 객체로 생성합니다. 
  + UserDetails 객체는 Security 내장 객체인 반면, userDetailsService는 UserDetailsService 내장 객체로부터 상속받은 UserDetailsServiceImpl 사용자 객체입니다. 이 객체 역시 아래에서 다루겠습니다.
* 4) 실제 인증 작업을 수행해서 이상유무를 판단 후, 이상이 없으면 Authentication을 생성합니다(setAuthentication). 이 부분은 전부 내장 객체에서 수행하는 동작이므로 추가 설명은 없습니다.

그렇다면 여기서 추가 생성해야 하는 객체를 살펴보겠습니다.

#### 4-2) Java Web Token(JWT) 클래스 생성

jwtToken 클래스는 앞서 위에 언급된 validateJwtToken() 메소드를 실행하기 위한 클래스로, 실제 이 클래스는 Java Web Token(JWT)을 가지고 수행하는 여러 가지 메소드를 정의하고 있습니다.
그러므로 validateJwtToken() 메소드 뿐만 아니라 다른 메소드도 같이 아래와 같이 나타내겠습니다.

{% highlight Java %}
@Component
public class JwtUtils {
	private static final Logger logger = LoggerFactory.getLogger(JwtUtils.class);

	@Value("${bezkoder.app.jwtSecret}")
	private String jwtSecret;

	@Value("${bezkoder.app.jwtExpirationMs}")
	private int jwtExpirationMs;

	public String generateJwtToken(Authentication authentication) {

		UserDetailsImpl userPrincipal = (UserDetailsImpl) authentication.getPrincipal();

		return Jwts.builder()
				.setSubject((userPrincipal.getUsername()))
				.setIssuedAt(new Date())
				.setExpiration(new Date((new Date()).getTime() + jwtExpirationMs))
				.signWith(SignatureAlgorithm.HS512, jwtSecret)
				.compact();
	}

	public String getUserNameFromJwtToken(String token) {
		return Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token).getBody().getSubject();
	}

	public boolean validateJwtToken(String authToken) {
		try {
			Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(authToken);
			return true;
		} catch (SignatureException e) {
			logger.error("Invalid JWT signature: {}", e.getMessage());
		} catch (MalformedJwtException e) {
			logger.error("Invalid JWT token: {}", e.getMessage());
		} catch (ExpiredJwtException e) {
			logger.error("JWT token is expired: {}", e.getMessage());
		} catch (UnsupportedJwtException e) {
			logger.error("JWT token is unsupported: {}", e.getMessage());
		} catch (IllegalArgumentException e) {
			logger.error("JWT claims string is empty: {}", e.getMessage());
		}

		return false;
	}
}
{% endhighlight %}

여기서 지역변수로 선언된 jwtSecret, jwtExpirationMs 변수는 application.properties에서 정의하고 있습니다. 참고하시기 바랍니다.
JWT를 검증하는 부분을 수행하기 때문에 추가 설명할 부분은 없습니다.

#### 4-3) UserDetail / UserDetailsService 객체 생성

위에서는 UserDetailsServiceImpl 객체를 선언하는 것으로 다루었는데, 실제로는 UserDetail 객체도 생성해야 합니다.
그 이유는 Service 구현 객체에서는 로그인 사용자 존재 여부 기능을 수행하고, 이에 대한 반환값인 UserDetail 역시 구현하기 위한 객체가 필요하기 때문입니다.

Service 구현 객체부터 보겠습니다.

{% highlight Java %}
@Service
public class UserDetailsServiceImpl implements UserDetailsService {
	@Autowired
	UserRepository userRepository;

	@Override
	@Transactional
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		User user = userRepository.findByUsername(username)
				.orElseThrow(() -> new UsernameNotFoundException("User Not Found with username: " + username));

		return UserDetailsImpl.build(user);
	}
}
{% endhighlight %}

위에 설명한대로 return값으로 UserDetailsImpl 객체의 build() 메소드를 실행합니다. 즉 반환되는 형태는 UserDetails 객체이지만 실제로는 UserDetailsImpl 객체를 사용하며,
이를 통해서 알 수 있는 것은 UserDetailsImpl 객체 역시 UserDetailsServiceImpl과 같이 상속받은 클래스의 형태로 구현된다라는 것을 짐작할 수 있을 것입니다.

{% highlight Java %}
public class UserDetailsImpl implements UserDetails {
	private static final long serialVersionUID = 1L;

	private Long id;

	private String username;

	private String email;

	@JsonIgnore
	private String password;

	private Collection<? extends GrantedAuthority> authorities;

	public UserDetailsImpl(Long id, String username, String email, String password,
			Collection<? extends GrantedAuthority> authorities) {
		this.id = id;
		this.username = username;
		this.email = email;
		this.password = password;
		this.authorities = authorities;
	}

	public static UserDetailsImpl build(User user) {
		List<GrantedAuthority> authorities = user.getRoles().stream()
			.map(role -> new SimpleGrantedAuthority(role.getName().name()))
			.collect(Collectors.toList());

		return new UserDetailsImpl(
			user.getId(), 
			user.getUsername(), 
			user.getEmail(),
			user.getPassword(), 
			authorities);
	}

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		return authorities;
	}

  ... Getter / Setter ...

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}

	@Override
	public boolean equals(Object o) {
		if (this == o)
			return true;
		if (o == null || getClass() != o.getClass())
			return false;
		UserDetailsImpl user = (UserDetailsImpl) o;
		return Objects.equals(id, user.id);
	}
}
{% endhighlight %}

Getter/Setter 선언부도 있는데, 이는 생략하였습니다.

코드를 보면, UserDetailsImpl 객체는 UserDetails라는 Security 내장 객체로부터 상속받은 클래스이며,
대부분이 Override한 것을 확인할 수 있습니다.
그리고 build() 메소드를 보면 user 객체의 정보를 가지고 UserDetailsImpl 객체를 생성한 것을 확인할 수가 있습니다.

이렇게 해서 생성된 UserDetailsImpl 객체를 통해서 웹페이지에 대한 Request가 들어오면 로그인 여부를 먼저 체크를 한다는 것을 알 수 있습니다.


#### 4-4) Request Filter의 사용

Request Filter는 모든 Request에 대한 처리에 사용됩니다.
그러나 모든 페이지가 로그인이 되어야만 이용 가능한 것은 아닙니다.

대표적으로 회원가입/로그인 페이지의 경우는 로그인이 되지 않은 상태에서도 접근이 가능하며, 이에 대한 부분은 
WebSecurityConfig의 configure()에서 이미 설정되어 있기 때문에 그것에 맞춰서 로그인 여부에 따른 결과를 반환하는 형태로 이루어집니다.

그렇다면 이제 회원가입과 로그인을 구현해 보도록 하며, Controller에서부터 출발하겠습니다.


### 5. 회원가입

{% highlight Java %}
@PostMapping("/signup")
public ResponseEntity<?> registerUser(@Valid @RequestBody SignupRequest signUpRequest) {
  if (userRepository.existsByUsername(signUpRequest.getUsername())) {
    return ResponseEntity
    .badRequest()
    .body(new MessageResponse("Error: Username is already taken!"));
  }

  if (userRepository.existsByEmail(signUpRequest.getEmail())) {
    return ResponseEntity
    .badRequest()
    .body(new MessageResponse("Error: Email is already in use!"));
}

  // Create new user's account
  User user = new User(signUpRequest.getUsername(), 
      signUpRequest.getEmail(),
      encoder.encode(signUpRequest.getPassword()));

  Set<String> strRoles = signUpRequest.getRole();
  Set<Role> roles = new HashSet<>();

  if (strRoles == null) {
    Role userRole = roleRepository.findByName(ERole.ROLE_USER)
        .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
    roles.add(userRole);
  } else {
    strRoles.forEach(role -> {
      switch (role) {
      case "admin":
        Role adminRole = roleRepository.findByName(ERole.ROLE_ADMIN)
          .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
        roles.add(adminRole);

        break;
      case "mod":
        Role modRole = roleRepository.findByName(ERole.ROLE_MODERATOR)
          .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
        roles.add(modRole);

        break;
      default:
        Role userRole = roleRepository.findByName(ERole.ROLE_USER)
          .orElseThrow(() -> new RuntimeException("Error: Role is not found."));
        roles.add(userRole);
      }
    });
  }

  user.setRoles(roles);
  userRepository.save(user);

  return ResponseEntity.ok(new MessageResponse("User registered successfully!"));
}
{% endhighlight %}

위 코드에서 제시하는 흐름은 어떻게 되느냐. 순서대로 볼까요.

* 1) signUpRequest로부터 회원가입 정보를 받습니다.
* 2) userRepository에서 Username 존재여부를 검사합니다(existsByUsername)
* 3) 존재하지 않으면 User 객체를 신규로 생성합니다.
* 4) 권한(Role)도 생성합니다.
* 5) 권한이 있으면 해당 권한을 추가하고, 없으면 기본 권한을 부여합니다.
* 6) 회원정보를 저장하고(save), 완료 메시지를 뿌려줍니다.

이제 추가로 만들어야 할 것을 보겠습니다.

* SignUpRequest: 회원가입 시 웹페이지에서 입력한 Request 내용입니다.
* Role: JPA에서 사용할 Object Class입니다.
* roleRepository: JPA에서 사용할 Role Repository입니다.
* ERole: Role의 전체 목록을 나타내는 enum값입니다.
* MessageResponse: Request에 대한 Response를 Message 형태로 나타낸 것입니다.

위 5개는 Object Class와 JPA Repository Class입니다. 그렇기 때문에 JPA 선언부와 객체 속성으로만 구성되어 있다고 보셔도 좋습니다.
그러므로 별도 설명은 하지 않아도 될 것 같으며, 코드는 원문 글을 참고하시면 될 것 같습니다.


### 6. 로그인

로그인도 똑같이 Controller를 생성합니다. 로그인은 회원가입과는 달리 코드는 짧지만 절차가 꽤 복잡하니 순서대로 잘 보셔야 합니다.

{% highlight Java %}
@PostMapping("/signin")
public ResponseEntity<?> authenticateUser(@Valid @RequestBody LoginRequest loginRequest) {

  Authentication authentication = authenticationManager.authenticate(
    new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword()));

  SecurityContextHolder.getContext().setAuthentication(authentication);
  String jwt = jwtUtils.generateJwtToken(authentication);
  
  UserDetailsImpl userDetails = (UserDetailsImpl) authentication.getPrincipal();		
  List<String> roles = userDetails.getAuthorities().stream()
    .map(item -> item.getAuthority())
    .collect(Collectors.toList());

  return ResponseEntity.ok(new JwtResponse(jwt, 
    userDetails.getId(), 
    userDetails.getUsername(), 
    userDetails.getEmail(), 
    roles));
}
{% endhighlight %}

먼저 로그인 컨트롤러의 첫 번째 줄에서 수행하는 동작부터 다시 보겠습니다.

{% highlight Java %}
		Authentication authentication = authenticationManager.authenticate(
				new UsernamePasswordAuthenticationToken(loginRequest.getUsername(), loginRequest.getPassword()));
{% endhighlight %}

앞서 나타냈던 AuthTokenFilter 수행 동작과 거의 유사합니다.

* AuthTokenFilter가 Request로 넘어온 정보를 가지고 로그인 여부를 검사했으면,
* Authentication Manager에서는 입력된 Request 정보가 올바른지를 검사하는 것으로 볼 수 있습니다.

하지만 차이가 있다면, 

* AuthTokenFilter는 Token 보유 여부 및 실제 사용자 정보를 확인해서 로그인 여부를 검사하는 데 그치지만,
* Authentication Manager는 사용자 로그인 정보가 올바른지를 검사하고, 올바를 경우 Token을 생성하고 값을 넘겨주는 반면 틀릴 경우 인증 실패도 알려야 합니다.

이에 따라 로그인에서는 추가적인 기능이 더 있다고 보셔도 좋습니다.

Authentication Manager에서는 로그인 정보의 유효성을 authenticataionManager.authenticate() 메소드로 검사하며 Security 내장 기능으로 수행합니다.
이 때 위의 프로세스에 의해서 UserDetailsService를 호출하는 것은 AuthTokenFilter와 동일하다 볼 수 있으며, 비밀번호 검사를 하는 추가 작업인 PasswordEncoder도 추가로 호출합니다.

그렇게 해서 로그인이 올바를 경우에는 authentication을 생성하고 Token도 생성하겠죠.

하지만 로그인이 올바르지 않을 경우에 수행하는 동작을 Security에서는 AuthenticationEntryPoint에서 정의하고 있으며, 이를 사용자가 정의하여 나타내기도 합니다.
이러한 사용자 정의 오류 클래스로 AuthEntryPointJwt를 생성하며, 아래와 같이 나타냅니다.

{% highlight Java %}
@Component
public class AuthEntryPointJwt implements AuthenticationEntryPoint {

	private static final Logger logger = LoggerFactory.getLogger(AuthEntryPointJwt.class);

	@Override
	public void commence(HttpServletRequest request, HttpServletResponse response,
			AuthenticationException authException) throws IOException, ServletException {
		logger.error("Unauthorized error: {}", authException.getMessage());
		response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Error: Unauthorized");
	}
}
{% endhighlight %}

이 클래스를 사용하겠다라는 것은 WebSecurityConfig에서 환경설정이 되어 있다는 점도 참고하시면 더욱 좋을 것 같네요.

위와 같이 구현하면 회원가입/로그인 부분 기능은 어느 정도 구현이 된 것으로 볼 수 있습니다.

사용할 사용자 DB에 따라서 연동되는 객체나 JPA Repository가 약간씩 달라질 수는 있겠지만, 위 사항을 참고해서 개발하신다면 Security를 사용한 인증은 구현할 수 있으니 좋은 참고가 되었으면 좋겠습니다.
